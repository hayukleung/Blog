## Activity 状态
- 运行：Activity 位于返回栈栈顶
- 暂停：Activity 不位于栈顶，但是仍然可见或部分可见
- 停止：Activity 不位于栈顶，且完全不可见
- 销毁：Activity 从返回栈中移除

## Activity 生命周期回调
- `onCreate`
- `onStart` Activity 由不可见变为可见
- `onResume`
- `onPause`
- `onStop` Activity 由可见变为不可见
- `onDestroy`
- `onRestart`

### 完整生存期
`onCreate` - `onDestroy`

### 可见生存期
`onStart` - `onStop`

### 前台生存期
`onResume` - `onPause`

## Activity 生命周期

![](http://ooun8fyfu.bkt.clouddn.com/17-12-19/33901122.jpg)
