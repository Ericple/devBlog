+++
date = '2025-02-09T20:56:18+08:00'
draft = false
title = 'Openharmony源代码之NAPI_CALL'
series = ["OpenHarmony"]
series_order = 1
+++
## 系统代码

在OpenHarmony系统源码中，存在许多NAPI_CALL的宏调用，例如：

```cpp
napi_value NapiAtManager::GrantUserGrantedPermission(napi_env env, napi_callback_info info)
{
    ACCESSTOKEN_LOG_DEBUG(LABEL, "GrantUserGrantedPermission begin.");

    auto* context = new (std::nothrow) AtManagerAsyncContext(env); // for async work deliver data
    if (context == nullptr) {
        ACCESSTOKEN_LOG_ERROR(LABEL, "New struct fail.");
        return nullptr;
    }

    std::unique_ptr<AtManagerAsyncContext> contextPtr {context};
    if (!ParseInputGrantOrRevokePermission(env, info, *context)) {
        return nullptr;
    }

    napi_value result = nullptr;

    if (context->callbackRef == nullptr) {
        NAPI_CALL(env, napi_create_promise(env, &(context->deferred), &result));
    } else {
        NAPI_CALL(env, napi_get_undefined(env, &result));
    }

    napi_value resource = nullptr;
    NAPI_CALL(env, napi_create_string_utf8(env, "GrantUserGrantedPermission", NAPI_AUTO_LENGTH, &resource));

    NAPI_CALL(env, napi_create_async_work(
        env, nullptr, resource,
        GrantUserGrantedPermissionExecute, GrantUserGrantedPermissionComplete,
        reinterpret_cast<void *>(context), &(context->work)));

    NAPI_CALL(env, napi_queue_async_work_with_qos(env, context->work, napi_qos_default));

    ACCESSTOKEN_LOG_DEBUG(LABEL, "GrantUserGrantedPermission end.");
    contextPtr.release();
    return result;
}
```

## NAPI_CALL_BASE

NAPI_CALL展开后可得到另一个宏：

`#define NAPI_CALL(env, theCall) NAPI_CALL_BASE(env, theCall, nullptr)`

而NAPI_CALL_BASE的实现为：
```
#define NAPI_CALL_BASE(env, theCall, retVal)
    do
    {
        if ((theCall) != napi_ok)
        {
            GET_AND_THROW_LAST_ERROR((env));
            return retVal;
        }
    } while (0)
```

## 总结

**即NAPI_CALL对`theCall`(第二个参数)进行了一次包装,如果`theCall`被调用后的返回值不等于`napi_ok`，则抛出异常，并返回空指针**