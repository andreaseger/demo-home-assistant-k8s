# DEMO k8s setup for home-assistant

Disclaimer: code extracted to share, the original source works for my setup, no guarentee or support to make this work for anyone else.

## Context / Prerequisits

- originally deployed on a (single-node) microk8s setup
- https://fluxcd.io setup (not shown here)
- secrets here are just examples
  - actually using https://getsops.io + https://age-encryption.org for secrets (so I can check them into git) see https://fluxcd.io/flux/guides/mozilla-sops/ for how to set this up with fluxcd
  
## Content

- `flux/bjw-s.yaml` flux helm repository for the chart we're using later
- `flux/home-assistant.yaml` is the flux kustomization -> adapt the git source and path
- `apps/home-assistant` is the kustomization with includes all necessary stuff for home-assistant
  - release.yaml + values.yaml: helm release & values
    - I'm using this generic "app template" chart https://bjw-s.github.io/helm-charts/docs/app-template/ that makes use of their common library https://bjw-s.github.io/helm-charts/docs/common-library/
    - even without using that chart the values files is probably quite self explanatory in case you want to use something else or do it without helm
    - `main` pod with `home-assistant`
      - run in `hostNetwork` (~line 12)
      - PVC for `/config`
      - configuration.yaml from configmap -> `apps/home-assistant/hass-configuration.yaml`
      - secrets.yaml from secret -> 
    - `code` pod with `code-server`
      - PVC for `/config`
      - mounting the home assistant config PVC as `/data`
      - this allows easy manipulation of all the extra additional config for home-assistant
    - ingress for everything
    - in `apps/home-assistant/values.yaml` around line 105 you see an example on how to mount host bluetooth
      - may require making the home-assistant priviledged (see around line 14), realized I don't have a use-case for bluetooth at the moment so disabled this again
- extra:
  - `apps/home-assistant/addons/influxdb`
    - bitnami helm release for a influxdb instance (will need the helm repo setup elsewhere - not included here)
    - will need some config in home-assistant https://www.home-assistant.io/integrations/influxdb/#examples 
    - activate by uncommenting line 6 here `apps/home-assistant/kustomization.yaml`
- FYI these comments `# {"$imagepolicy": "flux-system:home-assistant:name"}` are from flux image automation
