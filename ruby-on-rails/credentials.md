# Credentials

## Credentials

For editors that fork and exit immediately, it's important to pass a `wait` flag, otherwise the credentials will be saved immediately with no chance to edit.

```text
$ EDITOR="mate --wait" bin/rails credentials:edit
$ EDITOR=vi bin/rails credentials:edit
```

