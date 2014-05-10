Mockito when/then 的使用
====
<div style="font-size: 12px; color: #888; width:100%;text-align:left;margin-bottom:10px;">
作者：[爱看书不识字](http://blog.loafer.gitpress.org/)，发布与2014年5月
</div>  
这篇主要介绍使用 when/then 来配置mock对象的各种行为。   

##配置mock对象方法调用返回值
    List<String> mockedList = mock(List.class);

    when(mockedList.get(0)).thenReturn("first");

    String value = mockedList.get(0);
    verify(mockedList).get(0);
    assertEquals(value, "first");
<br>
##另一种方式配置mock对象方法调用返回值
    List<String> mockedList = mock(List.class);

    doReturn("first").when(mockedList).get(0);

    String value = mockedList.get(0);
    assertEquals(value, "first");
<br>
##配置方法调用抛出异常
    List<String> mockedList = mock(List.class);

    when(mockedList.get(0)).thenThrow(new RuntimeException());

    mockedList.get(0);
<br>
##配置方法连续调用行为
    List<String> mockedList = mock(List.class);

    when(mockedList.add(anyString())).thenReturn(false).thenThrow(new RuntimeException());

    mockedList.add("hello");
    mockedList.add("world");//此调用将抛出异常
<br>
##配置监控普通对象（即非mock对象）行为
主要用于验证另一个类的某个特定方法的调用实现。如果是同一个类的不同方法则没有意义。请查看官方文档13. Spying on real objects内容    
    List<String> list = new ArrayList<>();
    List<String> spy = spy(list);

    //可选步骤，此处可以sub一些方法
    when(spy.size()).thenReturn(100);

    //以下调用都是真实调用ArrayList的add方法
    spy.add("one");
    spy.add("two");

    int size = spy.size();
    assertThat(size, equalTo(100));
<br>
##配置模拟mock对象调用真实对象的方法
    List<String> mockedList = mock(ArrayList.class);

    //当调用size方法时，触发ArrayList的size方法调用
    when(mockedList.size()).thenCallRealMethod();

    int size = mockedList.size();
    assertThat(size, equalTo(0));
<br>
##配置模拟方法调用自定义的Answer
    List<String> mockedList = mock(List.class);

    when(mockedList.get(anyInt())).thenAnswer(new Answer<Object>() {
        @Override
        public Object answer(InvocationOnMock invocation) throws Throwable {
            Object[] args = invocation.getArguments();
            Object mock = invocation.getMock();
            return "called with arguments: " + Arrays.asList(args);
        }
    });

    String value = mockedList.get(1);
    assertEquals(value, "called with arguments: [1]");

<br>    
以上例子的实现代码你可以在[我的GitHub mockito工程]()中找到。   

##参考
http://www.baeldung.com/mockito-verify   
http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html  
