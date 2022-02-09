First, run a app server application with label app=app and expose it internally in the cluster:

kubectl run  webserver --image=nginx --labels project=xelence,role=web --expose --port 80

kubectl run  appserver --image=nginx --labels project=xelence,role=app --expose --port 80 


First Validate that we can call the app from web. So run a temporary Pod with the label app=web and get a shell in the Pod:

kubectl run -l project=xelence,role=app --image=alpine --restart=Never --rm -i -t appserver
kubectl run -l project=xelence,role=web --image=alpine --restart=Never --rm -i -t webserver1

Then run 
wget -qO- --timeout=2 http://appserver

To Delete
k delete po webserver
k delete svc webserver

NetworkPolicies
    Deny all traffic
    Allow only web to accept traffic from public
    Allow app to accept traffic from web
    