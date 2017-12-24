## Fragment 运行状态
- 运行状态：满足下面条件
1. Fragment 所在 Activity 进入运行状态时
- 暂停状态：满足下面条件
1. Fragment 所在 Activity 进入暂停状态时
- 停止状态：满足下面条件之一
1. Fragment 所在 Activity 进入停止状态时
2. Fragment 调用 remove()、replace() 将 Fragment 从 Activity 中移除，且调用 commit() 之前调用了 addToBackStack()
- 销毁状态：满足下面条件之一
1. Fragment 所在 Activity 进入销毁状态时
2. Fragment 调用 remove()、replace() 将 Fragment 从 Activity 中移除，且调用 commit() 之前没有调用 addToBackStack()

## Fragment 生命周期回调方法
`onAttach`
`onCreate`
`onCreateView`
`onActivityCreated`
`onStart`
`onResume`
`onPause`
`onStop`
`onDestroyView`
`onDestroy`
`onDetach`