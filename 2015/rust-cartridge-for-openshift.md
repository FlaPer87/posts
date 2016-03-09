<!---
$"metadata"$
{
  "md": true,
  "title": "Rust's cartridge for OpenShift",
  "draft": false,
  "slug": "rust-cartridge-for-openshift",
  "tags": [
    "rust",
    "openshift"
]}
$"metadata"$
-->

Now that we're getting closer and closer and closer to the [Rust](http://www.rust-lang.org/) 1.0 release - May 15th, in case you missed the announcement - it becomes more and more important to create an ecosystem around the language. This ecosystem will allow for more people to adopt Rust, to build new amazing things with it and to contribute back. Luckily enough, the community has been doing an amazing job on this area, we've more than 2000 crates in [crates.io](https://crates.io/).

In order to contribute to this ecosystem and to make it easier for others to deploy rust apps, I've taken some time to hack an [OpenShift Cartridge](http://docs.openshift.org/origin-m4/oo_cartridge_developers_guide.html) for rust, which is surprisingly called [openshift-rust-cart](https://github.com/FlaPer87/openshift-rust-cart).

OpenShift Cartridge
===================

Do you know what an [OpenShift Cartridge](http://docs.openshift.org/origin-m4/oo_cartridge_developers_guide.html) is? Yes? Skip this section, No? That's fine, I'll explain it here.

In a nutshell, a cartridge is a set of scripts used for managing the software that is required to run your application. It can be anything you need. There are cartridges for languages - Python, Ruby, PHP - for databases - Postgres, MySQL, MongoDB - and even frameworks - RoR, Django, etc. 

Some of these cartridges are provided and maintained by the OpenShift team and others are maintained by community members. Anyone can create a cartridge and it's actually very simple. In fact, there are also tools that makes this process even simpler, like the [Cartridge Development Kit](https://github.com/smarterclayton/openshift-cdk-cart).

Using the cartridge
===================

To create an OpenShift app with this cartridge you just need to execute the following command:

    rhc create-app myrust https://cartreflect-claytondev.rhcloud.com/reflect?github=FlaPer87/openshift-rust-cart

The above will create an app called `myrust` for you, which will be based on the template provided by the cartridge itself. That's it, no further steps required. The only thing you need to do now is to hack on your amazing rust app. :)

The example app is based on [nickel.rs](http://nickel.rs/) and provides a [basic http server](https://github.com/FlaPer87/openshift-rust-cart/blob/master/template/src/main.rs). You can - and should - create more amazing things than this simple web server. Go and do it!

Credits
=======

This cartridge is based on [openshift-go-cart](https://github.com/smarterclayton/openshift-go-cart) so lots of credits go to it.


***Happy Hacking***