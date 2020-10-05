# client-go

## informer+(controller)



图中informer是一个processLoop的通知流程，代码层面不存在informer这个数据结构，代码中的informer是图中全部组件的总和即为client-go/tools/cache/controller中的interface。

一个controller中包含

```go
type Controller interface {
	Run(stopCh <-chan struct{})
	HasSynced() bool
	LastSyncResourceVersion() string
}

// Controller is a generic controller framework.
type controller struct {
	config         Config
	reflector      *Reflector // 这个是和api交互核心组件，启动时初始化
	reflectorMutex sync.RWMutex
	clock          clock.Clock
}

// 最后控制器被重命名为informer
// newInformer returns a controller for populating the store while also
// providing event notifications.
func newInformer(
	// ...
	clientState Store,
) Controller {
	// This will hold incoming changes. Note how we pass clientState in as a
	// KeyLister, that way resync operations will result in the correct set
	// of update/delete deltas.
    // DeltaFIFO也需要存储
	fifo := NewDeltaFIFO(MetaNamespaceKeyFunc, clientState)

	cfg := &Config{
		Queue:            fifo,
		// ...

        // 生成process逻辑：对线程安全的local cache进行CRUD，并回调用户订阅的逻辑
		Process: func(obj interface{}) error {
			// from oldest to newest // 正序遍历，先处理老事件，再处理新事件
			for _, d := range obj.(Deltas) {
				switch d.Type {
				case Sync, Added, Updated:
						// clientState.Get(d.Object) // CRUD
						// h.OnUpdate(old, d.Object) // callback回调注册的handler

						// clientState.Add(d.Object)
						// h.OnAdd(d.Object)
				case Deleted:
					// clientState.Delete(d.Object)
					// h.OnDelete(d.Object)
				}
			}
			return nil
		},
	}
	return New(cfg)
}


func (c *controller) Run(stopCh <-chan struct{}) {
	// 生成反射器，监听apiserver
	r := NewReflector(
        // ...
        c.config.Queue, // 使用控制器自身的DeltaFIFO新建Reflector
        // ...
    ) 
	var wg wait.Group
	defer wg.Wait()

    // 启动Reflector，Reflector会list-Watch apiserver，根据event CRUD更新DeltaFIFO
	wg.StartWithChannel(stopCh, r.Run) 
    
	// 启动controller的process流程，不断popFIFO，然后调用处理逻辑
	wait.Until(c.processLoop, time.Second, stopCh) 
}

func (c *controller) processLoop() {
	for {
		obj, err := c.config.Queue.Pop(PopProcessFunc(c.config.Process))
		// ...
	}
}
```







- 其他

```go
import (
    v1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)
```



```go
// type clientset

func (c *Clientset) CoreV1() corev1.CoreV1Interface

type CoreV1Interface interface {
    RESTClient() rest.Interface
    ComponentStatusesGetter
    ConfigMapsGetter
    EndpointsGetter
    EventsGetter
    LimitRangesGetter
    NamespacesGetter
    NodesGetter
    PersistentVolumesGetter
    PersistentVolumeClaimsGetter
    PodsGetter
    PodTemplatesGetter
    ReplicationControllersGetter
    ResourceQuotasGetter
    SecretsGetter
    ServicesGetter
    ServiceAccountsGetter
}

type PodsGetter interface {
    Pods(namespace string) PodInterface
}

type PodInterface interface {
    Create(*v1.Pod) (*v1.Pod, error)
    Update(*v1.Pod) (*v1.Pod, error)
    UpdateStatus(*v1.Pod) (*v1.Pod, error)
    Delete(name string, options *metav1.DeleteOptions) error
    DeleteCollection(options *metav1.DeleteOptions, listOptions metav1.ListOptions) error
    Get(name string, options metav1.GetOptions) (*v1.Pod, error)
    List(opts metav1.ListOptions) (*v1.PodList, error)
    Watch(opts metav1.ListOptions) (watch.Interface, error)
    Patch(name string, pt types.PatchType, data []byte, subresources ...string) (result *v1.Pod, err error)
    GetEphemeralContainers(podName string, options metav1.GetOptions) (*v1.EphemeralContainers, error)
    UpdateEphemeralContainers(podName string, ephemeralContainers *v1.EphemeralContainers) (*v1.EphemeralContainers, error)

    PodExpansion
}
```



- 打印pod

`func printPod`



