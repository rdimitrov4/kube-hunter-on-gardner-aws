1) Connect the local kubectl on current shell instance to the Gardner cluster with a pre-generated token.
export KUBECONFIG=$KUBECONFIG:/path/to/file/kubeconfig--promart--q-lab.yaml

2) Setup Istio on the cluster 
- Platform setup for Gardner: https://istio.io/latest/docs/setup/platform-setup/gardener/ (networking Calico for further networking policy testing)

- Install Istio using istioctl https://istio.io/latest/docs/setup/getting-started/

------- Download Istio

Go to the Istio release page to download the installation file for your OS, or download and extract the latest release automatically (Linux or macOS):
    $ curl -L https://istio.io/downloadIstio | sh -

The command above downloads the latest release (numerically) of Istio. You can pass variables on the command line to download a specific version or to override the processor architecture. For example, to download Istio 1.6.8 for the x86_64 architecture, run:
    $ curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.6.8 TARGET_ARCH=x86_64 sh -

Move to the Istio package directory. For example, if the package is istio-1.11.3:
    $ cd istio-1.11.3

The installation directory contains:
    Sample applications in samples/
    The istioctl client binary in the bin/ directory.

Add the istioctl client to your path (Linux or macOS):
    $ export PATH=$PWD/bin:$PATH
    
------- Install Istio

    $ istioctl install
    
- Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:
    $ kubectl label namespace default istio-injection=enabled
    namespace/default labeled


3) Check the doplyed resources on default, istio-system and kube-system namespaces
    kubectl get all; kubectl get all -n istio-system; kubectl get all -n kube-system

4) Get kube-hunter
    https://github.com/aquasecurity/kube-hunter

5) Deploy kube-hunter as a pod in the cluster

From the folder with the kube-hunter-main binary
    :~/Downloads/kube-hunter-main$ kubectl create -f job.yaml

Check the created POD's ID
    :~/Downloads/kube-hunter-main$ kubectl get pods | grep "kube-hunter"

Get the pod logs
    :~/Downloads/kube-hunter-main$ kubectl logs kube-hunter-7jssg
    
INFO kube_hunter.modules.report.collector Started hunting
INFO kube_hunter.modules.report.collector Discovering Open Kubernetes Services
INFO kube_hunter.modules.report.collector Found vulnerability "Read access to pod's service account token" in Local to Pod (kube-hunter-7jssg)
INFO kube_hunter.modules.report.collector Found vulnerability "CAP_NET_RAW Enabled" in Local to Pod (kube-hunter-7jssg)
INFO kube_hunter.modules.report.collector Found vulnerability "Access to pod's secrets" in Local to Pod (kube-hunter-7jssg)
INFO kube_hunter.modules.report.collector Found vulnerability "AWS Metadata Exposure" in Local to Pod (kube-hunter-7jssg)

Vulnerabilities
For further information about a vulnerability, search its ID in: 
https://avd.aquasec.com/
+--------+----------------------+----------------------+----------------------+----------------------+----------------------+
| ID     | LOCATION             | CATEGORY             | VULNERABILITY        | DESCRIPTION          | EVIDENCE             |
+--------+----------------------+----------------------+----------------------+----------------------+----------------------+
| KHV053 | Local to Pod (kube-  | Information          | AWS Metadata         | Access to the AWS    | cidr: 10.250.0.0/19  |
|        | hunter-7jssg)        | Disclosure           | Exposure             | Metadata API exposes |                      |
|        |                      |                      |                      | information about    |                      |
|        |                      |                      |                      | the machines         |                      |
|        |                      |                      |                      | associated with the  |                      |
|        |                      |                      |                      | cluster              |                      |
+--------+----------------------+----------------------+----------------------+----------------------+----------------------+
| None   | Local to Pod (kube-  | Access Risk          | CAP_NET_RAW Enabled  | CAP_NET_RAW is       |                      |
|        | hunter-7jssg)        |                      |                      | enabled by default   |                      |
|        |                      |                      |                      | for pods.            |                      |
|        |                      |                      |                      |     If an attacker   |                      |
|        |                      |                      |                      | manages to           |                      |
|        |                      |                      |                      | compromise a pod,    |                      |
|        |                      |                      |                      |     they could       |                      |
|        |                      |                      |                      | potentially take     |                      |
|        |                      |                      |                      | advantage of this    |                      |
|        |                      |                      |                      | capability to        |                      |
|        |                      |                      |                      | perform network      |                      |
|        |                      |                      |                      |     attacks on other |                      |
|        |                      |                      |                      | pods running on the  |                      |
|        |                      |                      |                      | same node            |                      |
+--------+----------------------+----------------------+----------------------+----------------------+----------------------+
| None   | Local to Pod (kube-  | Access Risk          | Access to pod's      | Accessing the pod's  | ['/var/run/secrets/k |
|        | hunter-7jssg)        |                      | secrets              | secrets within a     | ubernetes.io/service |
|        |                      |                      |                      | compromised pod      | account/ca.crt', '/v |
|        |                      |                      |                      | might disclose       | ar/run/secrets/kuber |
|        |                      |                      |                      | valuable data to a   | netes.io/serviceacco |
|        |                      |                      |                      | potential attacker   | ...                  |
+--------+----------------------+----------------------+----------------------+----------------------+----------------------+
| KHV050 | Local to Pod (kube-  | Access Risk          | Read access to pod's | Accessing the pod    | eyJhbGciOiJSUzI1NiIs |
|        | hunter-7jssg)        |                      | service account      | service account      | ImtpZCI6Il9rbFhrTUNr |
|        |                      |                      | token                | token gives an       | aVdGX1BLUG96SEJtd0NU |
|        |                      |                      |                      | attacker the option  | MEhJanhWVnBhcjJ0bDBF |
|        |                      |                      |                      | to use the server    | cDA1cTQifQ.eyJhdWQiO |
|        |                      |                      |                      | API                  | ...                  |
+--------+----------------------+----------------------+----------------------+----------------------+----------------------+

---

Delete the created kube-hunter pod
    :~/Downloads/kube-hunter-main$ kubectl delete -f job.yaml

Make sure the pod is deleted
    host@host:~/Downloads/kube-hunter-main$ kubectl get pods | grep "kube-hunter"
    host@host:~/Downloads/kube-hunter-main$
    host@host:~/Downloads/kube-hunter-main$ kubectl get pods
-----------------------------------------------------------------------------------------------------------------------------

Kube-Hunter can be deployed to other namespaces aswell for further vulnerability testing
