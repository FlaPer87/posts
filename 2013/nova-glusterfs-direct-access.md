<!---
$"metadata"$
{
  "md": true,
  "title": "Using libgfapi to access Glusterfs Volumes in Nova",
  "draft": false,
  "slug": "nova-glusterfs-direct-access",
  "tags": [
    "qemu",
    "glusterfs",
    "openstack",
    "nova"
  ]
}
$"metadata"$
-->

Recently, I co-authored a [patch](https://review.openstack.org/#/c/39498/) for Nova that allows for GlusterFS volumes to be accessed directly from Qemu using [libgfapi](https://github.com/gluster/glusterfs/tree/master/api). Previous to this patch, it was only possible to access glusterfs by mounting them using GlusterFS FUSE client.

## About QEMU and libgfapi

Recently, in version 1.3, QEMU introduced a GlusterFS block driver based on libgfapi.

Libgfapi is a POSIX-like C library shipped along with GlusterFS, which allows to access Gluster's volumes without passing through its FUSE client. This integration brings in some benefits but, the most relevant ones are:

* Performance improvements by removing FUSE's overhead.
* Reduce the number of steps required to get to GlusterFS

There's no special configuration needed to use this, as long as you have QEMU >=1.3 and GlusterFS>=3.4 you should be fine. This is an example of what you can do:

    qemu-img create gluster://$GLUSTER_HOST/$GLUSTER_VOLUME/images 5G


If you'd like to know more about this specific implementation, I suggest you to read this well-explained blog [post](http://raobharata.wordpress.com/2012/10/29/qemu-glusterfs-native-integration/) where all the details about GlusterFS's driver are explained.

## What changed in Nova?

Small changes were required in order to make this work.

### Configuration changes

A new configuration parameter - `qemu_allowed_storage_drivers` - was added to enable or disable this feature. Direct access is disabled by default - this means `LibvirtGlusterfsVolumeDriver` will mount gluster's volume - and can be enabled by adding 'gluster' to the new configuration parameter.

### Code Changes

The patch is quite small - pasted right bellow - and just required to modify libvirt's configuration object to let QEMU know it should use use a network device and access it using gluster's protocol.

    -        options = connection_info['data'].get('options')
    -        path = self._ensure_mounted(connection_info['data']['export'], options)
    -        path = os.path.join(path, connection_info['data']['name'])
    -        conf.source_type = 'file'
    -        conf.source_path = path
    +
    +        data = connection_info['data']
    +
    +        if 'gluster' in CONF.qemu_allowed_storage_drivers:
    +            vol_name = data['export'].split('/')[1]
    +            source_host = data['export'].split('/')[0][:-1]
    +
    +            conf.source_ports = [None]
    +            conf.source_type = 'network'
    +            conf.source_protocol = 'gluster'
    +            conf.source_hosts = [source_host]
    +            conf.source_name = '%s/%s' % (vol_name, data['name'])
    +        else:
    +            path = self._ensure_mounted(data['export'], data.get('options'))
    +            path = os.path.join(path, data['name'])
    +            conf.source_type = 'file'
    +            conf.source_path = path

## Using it

**NOTE:** From now on, I'll assume you've installed nova, cinder, GlusterFS, libvirt and all the required pieces (I suggest you to use devstack if you're just testing this feature).

    $ # Edit nova.conf and add gluster to the qemu_allowed_storage_drivers list.
    $ dd if=/dev/zero of=gluster-volume bs=1M count=4096
    $ mkfs.xfs -f gluster-volume

So far we created an XFS loop device. We now have to mount it and add it to glusterfs. Execute as **root**:

    $ mkdir /srv/brick2
    $ mount -o loop -t xfs gluster-volume /srv/brick2
    $ gluster volume create <ip>:/srv/brick2 testvol1
    $ gluster volume start testvol1
    $ # Edit /etc/cinder/glusterfs_shares and add "$GLUSTER_HOST:/testvol1"
    $ # Restart cinder-volume

We just created a directory for our brick and mounted our loop device there. Then we created a volume in glusterfs, started it and then we added it to the `glusterfs_shares` file, which is were all glusterfs "shares" are specified. It is important to restart Cinder's volume service so it can reload the `glusterfs_shares` file.

We can now create our volume and attach it to our running instance.

    $ cinder create 1 # Creates a 1GB volume
    $ nova volume-attach <vm> <vol> auto

If nothing bad happened - you know, Murphy - you should see a new device attached to your instance. You can verify this by either ssh'ing into the running instance and listing all available devices - $ ls /dev/vd* - or dumping libvirt instance's XML.

Enjoy!