Checking the Events of a VirtualServer and VirtualServerRoute Resources
After you create or update a VirtualServer resource, you can immediately check if the NGINX configuration for that resource was successfully applied by NGINX:

$ kubectl describe vs cafe
. . .
Events:
  Type    Reason          Age   From                      Message
  ----    ------          ----  ----                      -------
  Normal  AddedOrUpdated  16s   nginx-ingress-controller  Configuration for default/cafe was added or updated
Note that in the events section, we have a Normal event with the AddedOrUpdated reason, which informs us that the configuration was successfully applied.

Checking the events of a VirtualServerRoute is similar:

$ kubectl describe vsr coffee 
. . .
Events:
  Type     Reason                 Age   From                      Message
  ----     ------                 ----  ----                      -------
  Normal   AddedOrUpdated         1m    nginx-ingress-controller  Configuration for default/coffee was added or updated
