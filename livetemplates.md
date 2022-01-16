- [MsTest Initialize](#mstest-initialize)
- [Async MSTest Method:](#async-mstest-method)
- [MSTest Method:](#mstest-method)

## MsTest Initialize
- Shortcut:
  - mstinit
- Description:
  - MsTest Initialize
- Snippet:
```
[TestInitialize]
public void TestInitialize()
{
    $END$
}
```



## Async MSTest Method:
- Shortcut:
  - amst
- Description:
  - Async MSTest Method
- Snippet:
```
[TestMethod]
public async Task $MethodName$()
{
    // Arrange
    $END$
    // Act
    
    // Assert
    Assert.AreEqual(true,false);
}
```

## MSTest Method:
- Shortcut:
  - mst
- Description:
  - MSTest Method
- Snippet:
```
[TestMethod]
public void $MethodName$()
{
    // Arrange
    $END$
    // Act
    
    // Assert
    Assert.AreEqual(true,false);
}
```