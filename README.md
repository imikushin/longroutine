# longroutine
Long running go routines (if you absolutely need them) for Kubernetes controllers

## Usage

Add `longroutine.SingleStarter` as one of your reconciler fields. Start your go routines with `StartSingle()` method:

```go
package whatever

import (
	"github.com/dispatchframework/longroutine"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"
)

type reconciler struct {
	upgrader
	RoutineStarter longroutine.SingleStarter
}

type upgrader interface{
	Upgrade()
}

func NewReconciler() *reconciler {
	return &reconciler{
		RoutineStarter: longroutine.NewSingleStarter(),
	}
}

func (r *reconciler) Reconcile(req reconcile.Request) (reconcile.Result, error) {
	var upgradeID string	
	// skip ...	

	// starting a long running go routine, e.g. a ClusterAPI cluster upgrade
	// if a routine is already running for this upgradeID, the new one will not be started
	r.RoutineStarter.StartSingle(upgradeID, func() {
		r.upgrader.Upgrade()
	})

	// skip ...
	
	return reconcile.Result{}, nil
}
```
