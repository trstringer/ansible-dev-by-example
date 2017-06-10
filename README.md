# Ansible Development by Example

### Why?

Ansible is an awesome tool for configuration management. It is also a highly utilized one, and there are so many ways to contribute as a community.

### What is this?

There is no doubt that Ansible is a complex tool, with lots of inner-workings, yet it is easy to work with as an end user. But on the other end of that, contributing to Ansible with code can sometimes be a daunting task.

This documentation is a way to show step-by-step how to develop Ansible modules, both new module development as well as bug fixes and debugging.

## Environment setup

1. Clone the Ansible repository: `$ git clone https://github.com/ansible/ansible.git`
1. Change directory into the repository root dir: `$ cd ansible`
1. Create a virtual environment: `$ python3 -m venv venv` (or for Python 2 `$ virtualenv venv`. Note, this requires you to install the virtualenv package: `$ pip install virtualenv`)
1. Activate the virtual environment: `$ . venv/bin/activate`
1. Install development requirements: `$ pip install -r requirements.txt`

:bulb: Starting new development now? Create a new branch: `$ git checkout -b my-new-branch`. If you are planning on contributing back to the main Ansible repostiry, fork the Ansible repository into your own GitHub account and developing against that.

## New module development

If you are creating a new module that doesn't exist, you would start working on a whole new file. Here is an example:

1. Navigate to the directory that you want to develop your new module in. E.g. `$ cd cd lib/ansible/modules/cloud/azure/`
1. Create your new module file: `$ touch my_new_test_module.py`
1. Paste this simple into the new module file: (explanation below)
```python
#!/usr/bin/env python

from ansible.module_utils.basic import AnsibleModule

class MyModule:

    def __init__(self):
        self.module_args = dict(
            name=dict(type='str', required=True),
            new=dict(type='bool', required=False, default=False)
        )

        self.result = dict(
            changed=False,
            state=dict()
        )

        self.module = AnsibleModule(
            argument_spec=self.module_args,
            supports_check_mode=True
        )

        self.result['state'] = dict(
            original_message=self.module.params['name'],
            new_message='goodbye'
        )

        if self.module.params['new']:
            self.result['changed'] = True

        if self.module.params['name'] == 'fail me':
            self.module.fail_json(msg='You requested this to fail', **self.result)

        self.module.exit_json(**self.result)

def main():
    MyModule()

if __name__ == '__main__':
    main()
```
1. Create an arguments file with the following content: (explanation below)
```json
{
  "ANSIBLE_MODULE_ARGS": {
    "name": "hello",
    "new": true
  }
}
```
1. Ensure that you can directly execute your new module: `$ chmod 755 ./my_new_test_module.py`
1. Run your test module locally and directly: `$ ./my_new_testmodule.py /tmp/args.json`

This should be working output that resembles something like the following:

```
{"changed": true, "state": {"original_message": "hello", "new_message": "goodbye"}, "invocation": {"module_args": {"name": "hello", "new": true}}}
```

### Explanation of module code

1. This is something
1. This is another thing

### Explanation of arguments file

1. More stuff
1. Some more
