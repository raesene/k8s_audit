# k8s_audit


We're looking at what can be spoofed or is unreliable in Kubernetes audit logs

## Information sources

 - [Audit Type](https://github.com/kubernetes/kubernetes/blob/622509830c1038535e539f7d364f5cd7c3b38791/staging/src/k8s.io/apiserver/pkg/apis/audit/types.go#L29)
 

## Audit ID

Audit ID can be specified by the client, so it's not reliable.

```
curl -H 'Audit-ID: Lorem' http://127.0.0.1:8001/api/v1/pods/
```

This will result in the audit record having the AuditID `Lorem`.

```json
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"Lorem","stage":"RequestReceived","requestURI":"/api/v1/pods/","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1","172.18.0.1"],"userAgent":"curl/7.81.0","objectRef":{"resource":"pods","apiVersion":"v1"},"requestReceivedTimestamp":"2023-10-01T09:25:13.742237Z","stageTimestamp":"2023-10-01T09:25:13.742237Z"}
```

## Source IPs

You can add `X-Forwarded-For` headers to the request, and the audit log will contain them.

```bash
curl -H 'Audit-ID: Lorem' -H 'X-Forwarded-For: 8.8.8.8' http://127.0.0.1:8001/api/v1/pods/
```

This will result in 

```json
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"Lorem","stage":"ResponseComplete","requestURI":"/api/v1/pods/","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["8.8.8.8","127.0.0.1","172.18.0.1"],"userAgent":"curl/7.81.0","objectRef":{"resource":"pods","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2023-10-01T09:28:15.307641Z","stageTimestamp":"2023-10-01T09:28:15.313353Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
```

You can also use `X-Real-IP` headers

```bash
curl -H 'Audit-ID: Lorem' -H 'X-Forwarded-For: 8.8.8.8' -H 'X-Real-Ip: 1.1.1.1' http://127.0.0.1:8001/api/v1/pods/
```

results in 

```json
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"Lorem","stage":"ResponseComplete","requestURI":"/api/v1/pods/","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["8.8.8.8","127.0.0.1","1.1.1.1","172.18.0.1"],"userAgent":"curl/7.81.0","objectRef":{"resource":"pods","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2023-10-01T09:31:36.617125Z","stageTimestamp":"2023-10-01T09:31:36.620628Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
```