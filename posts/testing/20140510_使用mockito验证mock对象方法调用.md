使用Mockito验证对象方法调用
===
##验证方法调用
<p>
<code>
    //创建一个mock对象   
    List<String> mockedList = mock(List.class);

    //使用mock对象触发一个交互
    mockedList.clear();

    //验证方法clear()是否被调用
    verify(mockedList).clear();
    //验证方法size()是否被调用
    verify(mockedList).size();//因为没有调用过size()，所以会验证失败
</code>
</p>
