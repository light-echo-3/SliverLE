# 代码解析

## xdl动态获取函数指针
//art.cpp  
kbArt::load_symbols
```c++
static int load_symbols() {
  LOGV("ArtHelper", "start load_symbols");
  api_level = getAndroidApiLevel();
  const char *artPath = getLibArtPath();
//  void *handle = xdl_open( artPath,
//                           XDL_TRY_FORCE_LOAD);
  void *handle = xdl_open( artPath,
                          XDL_TRY_FORCE_LOAD);
  LOGV("ArtHelper", "handle is %p", handle);

  walk_stack = reinterpret_cast<WalkStack_t>(xdl_dsym(handle,
                                                      "_ZN3art12StackVisitor9WalkStackILNS0_16CountTransitionsE0EEEvb",
                                                      nullptr));
  if (walk_stack == nullptr) {
    return -1;
  }

  LOGV("ArtHelper", "walk_stack handle is %p", walk_stack);


  suspend_thread_by_thread_id =
      reinterpret_cast<SuspendThreadByThreadId_t>(xdl_dsym(handle,
                                                           "_ZN3art10ThreadList23SuspendThreadByThreadIdEjNS_13SuspendReasonEPb",
                                                           nullptr));
  if (suspend_thread_by_thread_id == nullptr) {
    return -1;
  }

  if (api_level>__ANDROID_API_Q__){ //TODO Android 11未支持
    suspend_thread_by_peer =
        reinterpret_cast<SuspendThreadByPeer_t>(xdl_dsym(handle,
                                                         "_ZN3art10ThreadList19SuspendThreadByPeerEP8_jobjectNS_13SuspendReasonEPb",
                                                         nullptr));
    if (suspend_thread_by_peer == nullptr) {
      //todo handle
    }
  }else{
    suspend_thread_by_peer_Q =
        reinterpret_cast<SuspendThreadByPeer_Q_t>(xdl_dsym(handle,
                                                           "_ZN3art10ThreadList19SuspendThreadByPeerEP8_jobjectbNS_13SuspendReasonEPb",
                                                           nullptr));
    if (suspend_thread_by_peer_Q == nullptr) {
      //todo handle

    }
  }

  resume = reinterpret_cast<Resume_t>(xdl_dsym(handle,
                                               "_ZN3art10ThreadList6ResumeEPNS_6ThreadENS_13SuspendReasonE",
                                               nullptr));
  if (resume == nullptr) {
    return -1;
  }

  pretty_method = reinterpret_cast<PrettyMethod_t>(xdl_dsym(handle,
                                                            "_ZN3art9ArtMethod12PrettyMethodEb",
                                                            nullptr));
  if (pretty_method == nullptr) {
    return -1;
  }

//  fetchState = reinterpret_cast<FetchState_t>(xdl_dsym(handle,
//                                                       "_ZN3art7Monitor10FetchStateEPKNS_6ThreadEPNS_6ObjPtrINS_6mirror6ObjectEEEPj",
//                                                       nullptr));

 StackVisitorGetMethod = reinterpret_cast<void* (*)(void*)>(xdl_dsym(handle,
                                                                    "_ZNK3art12StackVisitor9GetMethodEv",
                                                                    nullptr));

  getCpuMicroTime =
      reinterpret_cast<GetCpuMicroTime_t>(xdl_dsym(handle, "_ZNK3art6Thread15GetCpuMicroTimeEv",
                                                   nullptr));
//  if (fetchState == nullptr) {
//    return -1;
//  }
  return 1;
}
```