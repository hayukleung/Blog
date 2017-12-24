## Activity 动态添加 Fragment
```java
private void replaceFragment(Fragment fragment) {
  FragmentManager fragmentManager = getSupportFragmentManager();
  FragmentTransaction transaction = fragmentManager.beginTransaction();
  transaction.replace(R.id.fragment_stub, fragment);
  // 压入返回栈 transaction.addToBackStack(null);
  transaction.commit();
}
```

## 在 Activity 中获取 Fragment
```java
DemoFragment demoFragment = (DemoFragment) getFragmentManager().findFragmentById(R.id.fragment_demo);
```

## 在 Fragment 中获取 Activity
```java
DemoActivity demoActivity = (DemoActivity) getActivity();
```