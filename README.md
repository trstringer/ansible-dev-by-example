# Ansible Development by Example

### Why?

Ansible is an awesome tool for configuration management. It is also a highly utilized one, and there are so many ways to contribute as a community.

### What is this?

There is no doubt that Ansible is a complex tool, with lots of inner-workings, yet it is easy to work with as an end user. But on the other end of that, contributing to Ansible with code can sometimes be a daunting task.

This documentation is a way to show step-by-step how to develop Ansible modules, both new module development as well as bug fixes and debugging.

# Environment setup

1. Clone the Ansible repository: `$ git clone https://github.com/ansible/ansible.git`
1. Change directory into the repository root dir: `$ cd ansible`
1. Create a virtual environment: `$ python3 -m venv venv` (or for Python 2 `$ virtualenv venv`. Note, this requires you to install the virtualenv package: `$ pip install virtualenv`)
1. Activate the virtual environment: `$ . venv/bin/activate`
1. Install development requirements: `$ pip install -r requirements.txt`
1. Run the environment setup script for each new dev shell process: `$ . hacking/env-setup`

:zap: After the initial setup above, every time you are ready to start developing Ansible you should be able to just run the following from the root of the Ansible repo: `$ . venv/bin/activate && . hacking/env-setup`

:bulb: Starting new development now? Fixing a bug? Create a new branch: `$ git checkout -b my-new-branch`. If you are planning on contributing back to the main Ansible repostiry, fork the Ansible repository into your own GitHub account and developing against your new non-devel branch in your fork. When you believe you have a good working code change, submit a pull request to the Ansible repository.

# New module development

If you are creating a new module that doesn't exist, you would start working on a whole new file. Here is an example:

- Navigate to the directory that you want to develop your new module in. E.g. `$ cd lib/ansible/modules/cloud/azure/`
- Create your new module file: `$ touch my_new_test_module.py`
- Paste this simple into the new module file: (explanation in comments)
```python
#!/usr/bin/env python

from ansible.module_utils.basic import AnsibleModule

class MyModule:

    def __init__(self):
        # define the available arguments/parameters that a user can pass to
        # the module
        self.module_args = dict(
            name=dict(type='str', required=True),
            new=dict(type='bool', required=False, default=False)
        )

        # seed the result dict in the object
        # we primarily care about changed and state
        # change is if this module effectively modified the target
        # state will include any data that you want your module to pass back
        # for consumption, for example, in a subsequent task
        self.result = dict(
            changed=False,
            state=dict()
        )

        # the AnsibleModule object will be our abstraction working with Ansible
        # this includes instantiation, a couple of common attr would be the
        # args/params passed to the execution, as well as if the module
        # supports check mode
        self.module = AnsibleModule(
            argument_spec=self.module_args,
            supports_check_mode=True
        )

        # manipulate or modify the state as needed (this is going to be the
        # part where your module will do what it needs to do)
        self.result['state'] = dict(
            original_message=self.module.params['name'],
            new_message='goodbye'
        )

        # use whatever logic you need to determine whether or not this module
        # made any modifications to your target
        if self.module.params['new']:
            self.result['changed'] = True

        # during the execution of the module, if there is an exception or a
        # conditional state that effectively causes a failure, run
        # AnsibleModule.fail_json() to pass in the message and the result
        if self.module.params['name'] == 'fail me':
            self.module.fail_json(msg='You requested this to fail', **self.result)

        # in the event of a successful module execution, you will want to
        # simple AnsibleModule.exit_json(), passing the key/value results
        self.module.exit_json(**self.result)

def main():
    MyModule()

if __name__ == '__main__':
    main()
```

# Local/direct module testing

You may want to test the module on the local machine without targeting a remote host. This is a great way to quickly and easily debug a module that can run locally.

- Create an arguments file with the following content: (explanation below)
```json
{
  "ANSIBLE_MODULE_ARGS": {
    "name": "hello",
    "new": true
  }
}
```
- Ensure that you can directly execute your new module: `$ chmod 755 ./my_new_test_module.py`
- Run your test module locally and directly: `$ ./my_new_testmodule.py /tmp/args.json`

This should be working output that resembles something like the following:

```
{"changed": true, "state": {"original_message": "hello", "new_message": "goodbye"}, "invocation": {"module_args": {"name": "hello", "new": true}}}
```

# Playbook module testing

If you want to test your new module, you can now consume it with an Ansible playbook.

- Create a playbook in any directory: `$ touch testmod.yml`
- Add the following to the new playbook file
```yaml
---
- name: test my new module
  connection: local
  hosts: localhost

  tasks:
    - name: run the new module
      my_new_test_module:
        name: 'hello'
        new: true
      register: testout

    - name: dump test output
      debug:
        msg: '{{ testout }}'
```
- Run the playbook and analyze the output: `$ ansible-playbook ./testmod.yml`


### Explanation of arguments file

1. More stuff
1. Some more
