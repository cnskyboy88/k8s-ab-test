# AB Test Sample by Weight and Specific Header
## Prepare Services and Client

### Install Istio
> kubectl apply -f install/kubernetes/istio-demo.yaml

### Deploy 2 Version of Services
> istioctl kube-inject -f flask.istio.yaml | kubectl apply -f -

### Deploy The Client Who Use The Services
> istioctl kube-inject -f sleep.yaml | kubectl apply -f -

### Request The Service
> 1. kubectl exec -ti sleep-65d4c8c58b-qh9fx -c sleep bash
> 2. Test
```bash
for i in `seq 20`;do http --body http://flaskapp/env/version;done
```

## Random AB By Weights:
> https://istio.io/docs/tasks/traffic-management/traffic-shifting/


> 1. kubectl apply -f flaskapp-destionrule.yaml
> 2. kubectl apply -f flaskapp-default-vs-v2.yaml
> 3. Test
```bash
for i in `seq 20`;do http --body http://flaskapp/env/version;done
```

## Random AB By Weights:
> https://istio.io/docs/tasks/traffic-management/traffic-shifting/

### Key config
```yaml
http:
  - route:
    - destination:
        host: flaskapp
        subset: v1
      weight: 90
    - destination:
        host: flaskapp
        subset: v2
      weight: 10
```

## Select by Specific HTTP Header:

### Key config
```yaml
 hosts:
  - flaskapp
  http:
  - match:
    - headers:
        x-canary:
          exact: newversion
    route:
    - destination:
        host: flaskapp
        subset: v2
  - route:
    - destination:
        host: flaskapp
        subset: v1
```
>  1. kubectl apply -f flaskapp-default-vs-header.yaml
>  2. Test
```bash
#I want new version
bash-4.4# for i in `seq 20`;do curl -H 'x-canary:newversion' http://flaskapp/env/version;done
v2v2v2v2v2v2v2v2v2v2v2v2v2v2v2v2v2v2v2v2

#I DON'T want new version
bash-4.4# for i in `seq 20`;do curl -H 'x-canary:NOTnewversion' http://flaskapp/env/version;done
v1v1v1v1v1v1v1v1v1v1v1v1v1v1v1v1v1v1v1v1
```

## Monitor
> kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &

Then visit: http://localhost:3000