# init
```
kubebuilder init --domain tutorial.kubebuilder.io --repo github.com/denis-zakharov/kubebuilder-cron --project-name=kubebuilder-cron
```

A domain is an API domain. An API group corresponds to a subdomain.

# k8s api
- group: a collection of related functionality.
- version: each group has one or more versions.
- kind: each group-version contains one or more API types (Kind or a GroupVersionKind or a GVK).
- resource: a use of a Kind in the API (an object manifest or a GroupVersionResource or a GVR).

The resources are always lowercase, and by convention are the lowercase form of the Kind.

Each GVK corresponds to a given root Go type in a package.

The `Scheme` maps Go types to corresponding `GVK` types.

# adding a new api
```
kubebuilder create api --group batch --version v1 --kind CronJob
```

`groupversion_info.go` has a GV info 
`GroupVersion{Group: "batch.tutorial.kubebuilder.io", Version: "v1"}`.

`cronjob_types.go` has a `Kind: CronJob` Go types. The corresponding generator marker
is `+kubebuilder:object:root`:

- `metav1.TypeMeta`: kind and version.
- `metav1.ObjectMeta` metadata like name, labels and namespace.
- `Spec` is a desired state.
- K8s reconciles the desired state with an observed state on the cluster
  and any other external state, and then records what it observed as a `Status`.

`Spec` and `Status` are subjects for implementation.

The `object` generator generates an implementation of the `runtime.Object` interface
for us, which is the standard interface that all types representing Kinds must implement.

Finally, we add the Go types to the API group. This allows us to add the types in this
API group to any Scheme.

```go
func init() {
    SchemeBuilder.Register(&CronJob{}, &CronJobList{})
}
```

# defining an api
All serialized fields must be `camelCase`.

Type coversions:
- primitives
- numbers: `int32` and `int64`
- decimals: `resource.Quantity` (like `100m`, `100Mi`, `1M`).
- time: `metav1.Time`.

# controller

Each controller focuses on one *root* Kind but may interact with other Kinds.

A controller implements `sigs.k8s.io/controller-runtime/pkg/reconcile.Reconciler` interface:

```go
type Reconciler interface {
	// Reconciler performs a full reconciliation for the object referred to by the Request.
	// The Controller will requeue the Request to be processed again if an error is non-nil or
	// Result.Requeue is true, otherwise upon completion it will remove the work from the queue.
	Reconcile(context.Context, Request) (Result, error)
}
```

# make manifests