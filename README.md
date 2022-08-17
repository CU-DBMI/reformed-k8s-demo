# Reformed Kubernetes Deployment

This repo contains Kubernetes configuration for setting up a
[Reformed](https://github.com/davidlougheed/reformed) server on a target
Kubernetes cluster. Reformed wraps Pandoc, a tool for converting between
different document formats, with a RESTful API. More details can be found in
[the API docs for
Reformed](https://github.com/davidlougheed/reformed/blob/main/README.md).

## Requirements

You'll need access to a running Kubernetes cluster (Docker Desktop's k8s is a
decent place to start), and `kubectl` installed on your local machine.

## Simple Usage

Assuming you have `kubectl` installed and your desired context activated, simply
run the following to deploy Reformed:

```
kubectl apply -f ./reformed/app.k8.yaml
```

This will launch Reformed on the cluster, but in order to access it you'll need
to forward traffic to it from your local machine. You can set up a
port-forwarding proxy to the service like so:

```
kubectl port-forward services/reformed-service 8005:8000
```

Now you'll be able to access the service at port 8005 on your local machine,
e.g. http://localhost:8005/api/v1/formats.

### Ingress Notes

The ingress configuration at `./reformed/ingress.k8.yaml` assumes you have
[ingress-nginx](https://github.com/kubernetes/ingress-nginx) (i.e., the
community version of the NGINX ingress for k8s) installed in your cluster. If
you do want to use the ingress, first edit the ingress file to match your
deployment's hostname and then apply the file as you did `app.k8.yaml` before.

## Interacting with Reformed

Once Reformed is running, you can make queries against the API. Take a look at
`using_reformed.ipynb` for a toy example on how to convert a document using it.

Alternatively, if you have `curl` installed, you can use it to make a simple
request against the service. For example, we'll convert `README.md` from
Markdown to a PDF using the following code snippet:

```
# first, remove README.pdf so we can be sure we're getting a fresh copy
rm -f README.pdf

# now let's invoke the service via curl to convert the file to a pdf
curl -X POST -F 'document=@README.md' \
    'http://localhost:8005/api/v1/from/markdown/to/pdf' \
    > README.pdf

# did it work? let's open it! 
# (this will use your shell to launch the pdf, however that's configured on your machine)
open README.pdf

# if all went well, you should be looking at a PDF of this document!
```
