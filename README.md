# ansible/modules/system/svc.py

This is an attempt to extend the [svc module](https://github.com/ansible/ansible/blob/devel/lib/ansible/modules/system/svc.py) for managing daemontools by adding a subclass to control [nosh](https://jdebp.eu/Softwares/nosh) (and possibly other daemontools family toolsets).

The module as it is (https://github.com/ansible/ansible/commit/eb1214baad0cbe2c4b7304caebb9ae1c7dc0d8db) offers to subclass the `Svc` class with the help of the `_load_dist_subclass` function which checks for the `distro` parameter.

If activated, this mechanism fails because the expression to obtain the `distro` parameter is incorrect:

    def _load_dist_subclass(cls, *args, **kwargs):
        '''
        Used for derivative implementations
        '''
        subclass = None
    
        distro = kwargs['module'].params['distro'] # this assignment fails with a KeyError
    
        # get the most specific superclass for this platform
        if distro is not None:
            for sc in cls.__subclasses__():
                if sc.distro is not None and sc.distro == distro:
                    subclass = sc
        if subclass is None:
            subclass = cls
    
    return super(cls, subclass).__new__(subclass)

The following assignment works but is rather cryptic and there is presumably a more straightforward way to obtain the passed-in `distro` parameter:

    distro = args[0][0].params['distro]

The solution adopted here is to simplify the function signature, making it more readable, since only one argument is given to the Svc instance (`svc = Svc(module)`):

    def _load_dist_subclass(cls, module):
        ...
        distro = module.params['distro']

Subclasses can then define a `distro` attribute and the mechanism works.

## minor corrections

The `distro` parameter referred to in `_load_dist_subclass` is actually called `dist` in the AnsibleModule `argument_spec`

The `distro` parameter is not mentionned in the documentation.

The `killed` state mentionned in the documentation but not listed as possible state.

The state -> method correspondence has needless indirection.

## usage

    - svc:
        name: sshd
        state: started
        distro: nosh
