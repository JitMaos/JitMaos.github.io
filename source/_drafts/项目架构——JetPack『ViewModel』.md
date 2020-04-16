---
title: JetPack『ViewModel』
tags: [JetPack,ViewModel]
---

ViewModel被设计用于存储并管理UI相关的数据，因此可以在配置变化时用于保存UI相关的数据。

ViewModel对象的生命周期由传递至ViewModelProvider的LifeCycle所决定。

```java
MyViewModel viewmodel = ViewModelProviders.of(getActivity()).get(MyViewModel.class)
```

这意味着，当一个Activity关联多个Fragment时，可以使用Activity的LifeCycle创建ViewModel对象，并使用该ViewModel对象在Fragment之间实现数据共享。

```java
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}

public class MasterFragment extends Fragment {
    private SharedViewModel model;
    public void onActivityCreated() {
        model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends LifecycleFragment {
    public void onActivityCreated() {
        SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        model.getSelected().observe(this, { item ->
           // update UI
        });
    }
}
```

