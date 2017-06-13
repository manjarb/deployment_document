# Persistent environment variables (Ubuntu)

```sh
remote $ ~/.pam_environment
```

This file is specifically meant for setting a user's environment. It is not a script file, but rather consists of assignment expressions, one per line. This example sets the variable FOO to a literal string and modifies the PATH variable:
```sh
FOO=bar
PATH DEFAULT=${PATH}:/home/@{PAM_USER}/MyPrograms
```

Refenrences
https://help.ubuntu.com/community/EnvironmentVariables#Persistent_environment_variables
