## AlwaysAllow / AlwaysDeny

Pure API server flags.

KIND Config(YAML):
    authorization-mode: AlwaysAllow

    OR

    authorization-mode: AlwaysDeny

Behavior:

    ✔ AlwaysAllow → everything works
    ✔ AlwaysDeny → everything blocked

⚠ Testing / debugging only
⚠ Never production