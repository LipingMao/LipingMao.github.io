---
title: Openstack Lib - oslo.config
---
## Backgroup
> Reduce duplicated code in different openstack project.

## Basic Usage
> Here is a sample which help you have a basic and quick use oslo.config

**Basic Usage Sample:**

Create file named "app":

    #!/bin/python

    # import cfg from oslo_config
    from oslo_config import cfg

    # define options which you need to use
    opts = [cfg.StrOpt('test', default='helo world', help=('help messaage'))]
    CONF = cfg.CONF

    # register opts in group, if the group does not existed, it will help you auto create
    CONF.register_opts(opts, group='limao')

    if __name__ == '__main__':
        # oslo_config will find your config in /etc/{project}/{name_of_process}.conf
        # if you did not have default_config_files here.
        CONF(project='limao')
        print 'CONF.limao.test %s' % CONF.limao.test


Create a config file in /etc/limao/app.conf:

    [limao]
    test = 'Real world'


Test code:

    # python app
    CONF.limao.test Real world


## Refer Doc
1. [oslo.config developer doc](http://docs.openstack.org/developer/oslo.config/cfg.html)

