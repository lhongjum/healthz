<p align="center">
  <img width="26%" src="https://user-images.githubusercontent.com/22690219/119139792-1b1b8000-ba4c-11eb-8c88-34c439eada3b.png" />
</p>
 
[![Go Report Card](https://goreportcard.com/badge/github.com/bh90210/healthz)](https://goreportcard.com/report/github.com/bh90210/healthz)
[![codecov](https://codecov.io/gh/bh90210/healthz/branch/main/graph/badge.svg?token=9PSK4W6VJ9)](https://codecov.io/gh/bh90210/healthz)


# healthz

A tiny & extremely simple to use library for Kubernetes liveness/readiness/termination checks.

# Use

_For full examples see the [examples]() folder._

## Start

In your `init` or at the begging of your `main` function include:
```go
hc := healthz.NewCheck("live", "ready", "8080")

go func() {
	if err := hc.Start(); err != nil {
		// do some error handling
	}
}()
```
_It is a blocking function._

Defining `healthz.NewCheck()` arguments is optional. If nothing's set ie. `healthz.NewCheck("", "", "")` the above default values will be used.

Kubernetes config excerpt:
```yaml
...
        livenessProbe:
          httpGet:
            path: /live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 60
          timeoutSeconds: 5
          failureThreshold: 6
          successThreshold: 1
        startupProbe:
          httpGet:
            path: /live
            port: 8080
          failureThreshold: 15
          periodSeconds: 5
...
```

## Ready & NotReady

### Ready

The default state is `NotReady`. When your application is ready to service requests you should set it to:
```go
hc.Ready()
``` 

### NotReady

If your application stops being able to server requests and you wish to let Kubernetes proceed to the appropriate action based on `restartPolicy` (either wait or restart the container) you can achieve that by setting:
```go
hc.NotReady()
```

Kubernetes config excerpt:
```yaml
...
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 5
          failureThreshold: 6
          successThreshold: 1
...
```

## Terminating

Kubernetes allows a grace period for any necessary clean up based on your `terminationGracePeriodSeconds` config setting.
```go
if term := hc.Terminating(); term == true {
	// do some clean up
}
```
_It is a blocking function._

Kubernetes config excerpt:
```yaml
...
      terminationGracePeriodSeconds: 5
...
```

## gRPC

```go
hc.GRPC()
```

Kubernetes config excerpt:
```yaml
...
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 5
          failureThreshold: 6
          successThreshold: 1
...
```
# Contributing

We are using a feature request workflow. Fork the repo create a new branch ie `fix/http` or `feat/newfeature` and make a PR against `main` branch.

# References

1. https://kubernetes.io/blog/2018/10/01/health-checking-grpc-servers-on-kubernetes/
2. https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
3. https://github.com/grpc-ecosystem/grpc-health-probe/
4. https://pkg.go.dev/google.golang.org/grpc/health/grpc_health_v1?utm_source=godoc
5. https://developers.redhat.com/blog/2020/11/10/you-probably-need-liveness-and-readiness-probes#example_4__putting_it_all_together