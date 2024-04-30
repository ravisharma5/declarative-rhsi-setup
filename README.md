# declarative-rhsi-setup
Repo with instructions to set Red Hat Service Interconnect and manage the sites declaratively

## Setup Red Hat Service Interconnect(RHSI) on OpenShift using operator framework

Install the latest RHSI operator.
```bash
oc apply -f subscription-rhsi.yaml
```

Setup Webterminal to help watch the skupper pods and generate required tokens for connect other sites
```bash
oc apply -f subscription-web-terminal.yaml
```

If Skupper is not installed as part of your Webterminal toolkit then set the image to the below image within the terminal window.

```bash
wtoctl set image quay.io/redhatintegration/rhi-tools:dev3
```
Above will restart the terminal window on the OpenShift console and set the image which has Skupper pre installed on it. Feel free to update the version of it if required.

