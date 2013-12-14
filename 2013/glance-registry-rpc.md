<!---
$"metadata"$
{
  "md": true,
  "title": "Glance Registry Driver",
  "draft": false,
  "slug": "glance-registry-driver",
  "tags": [
    "openstack",
    "glance",
    "python"
  ]
}
$"metadata"$
-->

A lot is going on on Glance lately, although we don't speak much about it. We've been working on several blueprints, and most of them aim to promote glance as a public service. Some of those blueprints are:

* [multiple-image-locations](https://blueprints.launchpad.net/glance/+spec/multiple-image-locations)
* [glance-cinder-driver](https://blueprints.launchpad.net/glance/+spec/glance-cinder-driver)
* [locations-policy](https://blueprints.launchpad.net/glance/+spec/locations-policy)
* [registry-api-v2](https://blueprints.launchpad.net/glance/+spec/registry-api-v2)
* [registry-db-driver](https://blueprints.launchpad.net/glance/+spec/registry-db-driver)

I'd like to say a few words about the blueprint I've been working on for H-2, the Registry DB Driver. It is more like bringing back an old feature than implementing a new one, but it changed radically.

Motivation
==========

Before going any further, let me spend a few words explaining what the registry API v2 does and why this driver is needed.

Glance's API service v1 requires a registry service to be up and running, this service is responsible for all operations that access the database. This was the default and unique architecture Glance supported for a long time. Later on, when glance-api v2 was developed, this behavior changed and no registry service was required anymore but, legacy deployments - the ones based on a registry service - were left uncovered and it was then required to put database credentials in the public API service.

In order to address and support such legacy deployments, and avoid putting database credentials into glance-api's configuration, we worked on a new glance-registry API - registry's v2 -  that implements a very simple RPC protocol over HTTP. The whole idea was to be able to use this service as a proxy to access the database without putting its credentials into the public service configuration file.

Since glance-api v2 is based on a well-defined domain model, the only way to support this glance-registry's API v2 was creating a new database driver based on Glance's database API, which, instead of accessing the database directly, would forward those calls to the remote RPC Registry service that would then execute the actual call to the database.

Implementation
==============

The whole implementation happened in 2 blueprints. [The first one](https://blueprints.launchpad.net/glance/+spec/registry-api-v2) implemented the RPC protocol over HTTP and [the second one](https://blueprints.launchpad.net/glance/+spec/registry-db-driver) implemented the database driver that talks to glance-registry.

The RPC protocol is very simple, it receives and sends JSON from / to the client. It allows the client to call any of the public - as not prefixed with _ - functions registered in the server side and it also propagates exceptions back to the client as long as their namespace is listed in the `allowed_rpc_exception_modules` configuration parameter. You can check the RPC code [here](https://github.com/openstack/glance/blob/master/glance/common/rpc.py#L79).

The database driver for the registry service was surprisingly simple to implement. Since the registry service can now act as a proxy for the registered functions - thanks to the fact that all public functions of the database driver are exposed through the RPC protocol - the database driver implementation just needs to forward those function calls to the registry service and send the response back to the caller. The implementation can be found [here](https://github.com/openstack/glance/tree/master/glance/db/registry)

RPC Server
==========

As mentioned before, the RPC protocol is pretty simple. It requires methods or functions to be registered and at every call it walks through the commands received, looks up for that command in the `registered` dictionary and calls its function. This is the piece of code were magic happens:

    for cmd in commands:
        # kwargs is not required
        command, kwargs = cmd["command"], cmd.get("kwargs", {})
        method = self._registered[command]
        try:
            result = method(req.context, **kwargs)
        except Exception as e:
            if self.raise_exc:
                raise

            cls, val = e.__class__, str(e)
            msg = (_("RPC Call Error: %(val)s\n%(tb)s") %
                   dict(val=val, tb=traceback.format_exc()))
            LOG.error(msg)

            # NOTE(flaper87): Don't propagate all exceptions
            # but the ones allowed by the user.
            module = cls.__module__
            if module not in CONF.allowed_rpc_exception_modules:
                cls = exception.RPCError
                val = str(exception.RPCError(cls=cls, val=val))

            cls_path = "%s.%s" % (cls.__module__, cls.__name__)
            result = {"_error": {"cls": cls_path, "val": val}}
        results.append(result)
    return results


Database API
============

Not everything was as straightforward as it seems. During the implementation of the database driver, I faced some issues caused by some design flaw we had in Glance.

The one I care most about required me to change some of Glance's database API functions. Turns out that the database API wasn't as abstract as it should be and some of its function's arguments depended on back-end specific objects. This raised awful exceptions when the registry client tried to serialize those objects before sending them to the registry service.

Other things were changed in the API in order to keep consistency throughout it. Changes can be found [here](https://review.openstack.org/#/c/36218/).

Architecture
============

Although it is recommended to let glance-api talk to the database directly - mostly performance wise - it is now possible to use the registry service for that task.

If you were to deploy glance and use the registry service, it is recommended to have multiple instances of it based on how many glance-api instance you have. Using the new registry's API will require more HTTP calls than the old one did since it now executes 1 call for each database access the glance-api service does.