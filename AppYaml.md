# Configuring `app.yaml` #

In order to use Fantasm, you must provide a mapping to it in your `app.yaml`. In the appropriate place (likely at the top of your `handlers` section), add the following lines:

```
- url: /fantasm/.*
  script: fantasm/main.py
  login: admin
```

The `url` must match the `root_url` as configured in your [fsm.yaml](YamlDescription#Top-level_YAML_attributes.md) file.

The `login: admin` is not strictly required, but without it, be aware that the general public can invoke your machines and see your Fantasm console (coming soon!). Because the Fantasm url structure is regular, it is possible for you to configure some machines to be publicly accessible while others are protected. Use at your own peril!