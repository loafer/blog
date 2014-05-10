使用Mockito验证对象方法调用
===

<div style="font-size: 12px; color: #888; width:100%;text-align:left;margin-bottom:10px;">作者:[爱看书不识字](http://blog.loafer.gitpress.org/)，发布与2014年5月</div>    
##验证mock对象方法调用

    //创建一个mock对象   
    List<String> mockedList = mock(List.class);

    //使用mock对象触发一个交互
    mockedList.clear();

    //验证方法clear()是否被调用
    verify(mockedList).clear();
    //验证方法size()是否被调用
    verify(mockedList).size();//因为没有调用过size()，所以会验证失败 
<!--more-->
<br>   
##验证mock对象方法调用次数
    List<String> mockedList = mock(List.class);

    mockedList.clear();
    mockedList.clear();
    mockedList.size();

    //验证方法size被执行过1次
    verify(mockedList).size();
    //验证方法size被执行过1次
    verify(mockedList, times(1)).size();
    //验证方法size至少被被执行过1次
    verify(mockedList, atLeastOnce()).size();

    //验证方法clear被执行过2次
    verify(mockedList, times(2)).clear();
    //验证方法clear至少被执行过1次
    verify(mockedList, atLeastOnce()).clear();
    //验证方法clear至少被执行过2次
    verify(mockedList, atLeast(2)).clear();
    //验证方法clear最多被执行过2次
    verify(mockedList, atMost(2)).clear();   

<br>   
##验证mock对象没有方法调用
    List<String> mockedList = mock(List.class);

    //验证mockedList没有发生过交互
    verifyZeroInteractions(mockedList);   
 
<br>       
##验证mock对象是否存在多余方法调用
    List<String> mockedList = mock(List.class);

    mockedList.size();
    mockedList.clear();

    //验证方法size是否被调用
    verify(mockedList).size();
    //检查是否存在未验证的方法调用，存在则验证失败
    verifyNoMoreInteractions(mockedList); //验证失败，因为没有验证方法clear调用   

<br>     
##验证mock对象方法执行顺序
    //验证单个mock对象方法的执行顺序
    List<String> mockedList = mock(List.class);

    mockedList.add("one");
    mockedList.size();
    mockedList.clear();

    //注意：如果以上调用顺序没有按照下面的验证顺序执行，
    //则验证失败
    InOrder inOrder1 = inOrder(mockedList);
    inOrder1.verify(mockedList).add("one");
    inOrder1.verify(mockedList).size();
    inOrder1.verify(mockedList).clear();


    //验证多个mock对象的调用顺序
    List<String> firstMock = mock(List.class);
    List<String> secondMock = mock(List.class);

    firstMock.add("was called first");
    secondMock.add("was called second");

    //注意：如果以上调用顺序没有按照下面的验证顺序执行，
    //则验证失败
    InOrder inOrder2 = inOrder(firstMock, secondMock);
    inOrder2.verify(firstMock).add("was called first");
    inOrder2.verify(secondMock).add("was called second");   

<br>   
##验证方法调用是否使用了指定的参数
    List<String> mockedList = mock(List.class);

    mockedList.add("one");

    //验证方法add调用时传入的参数是“one”
    verify(mockedList).add("one"); //验证成功
    //验证方法add调用时传入的参数是“two”
    verify(mockedList).add("two"); //验证失败  

<br>   
##使用Argument Matcher参数验证方式调用
    List<String> mockedStringList = mock(List.class);
    List<Integer> mockedIntegerList = mock(List.class);

    mockedStringList.add("one");
    mockedIntegerList.add(999);

    //验证mockedStringList的方法add被调用，并且参数类型是String
    verify(mockedStringList).add(anyString());
    //验证mockedIntegerList的方法add被调用，并且参数类型是Integer
    verify(mockedIntegerList).add(anyInt());   

<br>        
##使用ArgumentCaptor参数验证方法调用
    List<String> mockedList = mock(List.class);

    mockedList.addAll(Lists.<String>newArrayList("someElement"));
    ArgumentCaptor<List> argumentCaptor = ArgumentCaptor.forClass(List.class);

    //检查方法addAll是否被调用
    verify(mockedList).addAll(argumentCaptor.capture());
    //检查方法addAll调用时的参数中是否含有"someElement"
    List<String> capturedArgument = argumentCaptor.getValue();
    assertThat(capturedArgument, hasItem("someElement"));  

以上例子的实现代码你可以在[我的GitHub mockito工程]()中找到。  

##参考
http://www.baeldung.com/mockito-verify   
http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html   