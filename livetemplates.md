- [1. MsTest Initialize](#1-mstest-initialize)
- [2. Async Exception Delegate](#2-async-exception-delegate)
- [3. Async MSTest Method:](#3-async-mstest-method)
- [4. MSTest Method:](#4-mstest-method)

## 1. MsTest Initialize
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

## 2. Async Exception Delegate
- Shortcut:
  - msaexcp
- Description:
  - An async func delegate which can be used to assert thrown exceptions
- Snippet:
```
Func<Task> act = async () =>
{
    await $END$
};
```



## 3. Async MSTest Method:
- Shortcut:
  - amst
- Description:
  - Async MSTest Method
- Snippet:
```
[TestMethod]
public async Task $MethodName$_Should$Behaviour$_When$Condition$()
{
    // Arrange
    $END$
    // Act
    
    // Assert
    Assert.AreEqual(true,false);
}
```

## 4. MSTest Method:
- Shortcut:
  - mst
- Description:
  - MSTest Method
- Snippet:
```
[TestMethod]
public void $MethodName$_Should$Behaviour$_When$Condition$()
{
    // Arrange
    $END$
    // Act
    
    // Assert
    Assert.AreEqual(true,false);
}
```
