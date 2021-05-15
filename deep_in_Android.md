# binder

## sample code

**binder_native_client_test.cpp**

```c++
#include "binder_native_server.h"

int main()
{
    // 1. get service manager
    sp <IServiceManager> sm = defaultServiceManager();

    // 2. get HelloService
    sp <IBinder> binder = sm->getService(string16("service.helloService"));
    sp <IHelloNativeService> cs = interface_cast <IHelloNativeService> (binder);

    // 3. call into server with remote method
    cs->sayHello();

    return 0;
}
```

**binder_native_server_proc.cpp**

```c++
#include "binder_native_server.h"

int main(void)
{
    // 1. Get servcie manager
    sp <IServiceManager> sm = defaultServiceManager();

    // 2. add Hello service
    sm->addService(Sring16("service.helloService"), new BnHelloNativeService());

    // 3. start thread pool and add main thread to it.
    ProcessState::self()->startThreadPool();
    IPCThreadState::self()->joinThreadPool();
}
```

**binder_native_server.h**

```c++
#include "binder_native_server.h"

int main(void)
{
    // 1. Get servcie manager
    sp <IServiceManager> sm = defaultServiceManager();

    // 2. add Hello service
    sm->addService(Sring16("service.helloService"), new BnHelloNativeService());

    // 3. start thread pool and add main thread to it.
    ProcessState::self()->startThreadPool();
    IPCThreadState::self()->joinThreadPool();
}
```

**binder_native_server.cpp**

```c++
#include "binder_native_server.h"

namespace android {
    // 1. use macro to implement service
    IMPLEMENT_META_INTERFACE(HelloNativeService, "android.demo.helloNativeService");

    // 2. client
    BpHelloNativeService::BpHelloNativeService(const sp<IBinder>& impl): BpInterface<IHelloNativeService>(impl) {}

    void BpHelloNativeService::sayHello() {
        cout << "BpHelloNativeService::sayHello" << endl;
        Parcel data, reply;
        // 3. specify the binder destnition
        data.writeInterfaceToken(IHelloNativeService::getInterfaceDescriptor());
        // 4. transact CODE_HELLO to destnition process
        remote()->transact(CODE_HELLO, data, &reply);
    }

    // 5. server
    status_t BnHelloNativeService::OnTransact(uint_t code, const Parcel& data, Parcel* reply, uint32_t flags) {
        switch (code) {
        case CODE_HELLO:
            cout << "BnHelloNativeService::OnTransact CODE_HELLO" << endl;
            CHECK_INTERFACE(IHelloNativeService, data, reply);
            // 6. finally say hello
            sayHello();
            reply->writeInt32(2021);
            return NO_ERROR;
        default:
            break;
        }
        return NO_ERROR;
    }

    void BnHelloNativeService::sayHello() {
        cout << "BnHelloNativeService::sayHello" << endl;
    }
}
```

