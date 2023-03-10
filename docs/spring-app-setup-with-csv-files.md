# 在 Spring 应用程序中通过 CSV 外部化设置数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-app-setup-with-csv-files>

## 1。概述

在本文中，我们将**使用 CSV 文件**将应用程序的设置数据具体化，而不是硬编码。

该设置过程主要涉及在新系统上设置新数据。

## 2。一个 CSV 库

让我们首先介绍一个简单的用于 CSV 的库——[Jackson CSV 扩展](https://web.archive.org/web/20220627093349/https://github.com/FasterXML/jackson):

```java
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-csv</artifactId>       
    <version>2.5.3</version>
</dependency>
```

当然，在 Java 生态系统中有许多可用的库来处理 CSV。

我们在这里使用 Jackson 的原因是——Jackson 可能已经在应用程序中使用了，我们读取数据所需的处理相当简单。

## 3。设置数据

不同的项目需要设置不同的数据。

在本教程中，我们将设置用户数据——基本上是**用一些默认用户**准备系统。

下面是包含用户的简单 CSV 文件:

```java
id,username,password,accessToken
1,john,123,token
2,tom,456,test
```

注意文件的第一行是如何成为标题行的——列出每一行数据中的字段名称。

## 3。CSV 数据加载器

让我们首先创建一个简单的数据加载器来**将数据从 CSV 文件读取到工作内存**中。

### 3.1。加载对象列表

我们将实现`loadObjectList()`功能，从文件中加载特定`Object`的完全参数化列表:

```java
public <T> List<T> loadObjectList(Class<T> type, String fileName) {
    try {
        CsvSchema bootstrapSchema = CsvSchema.emptySchema().withHeader();
        CsvMapper mapper = new CsvMapper();
        File file = new ClassPathResource(fileName).getFile();
        MappingIterator<T> readValues = 
          mapper.reader(type).with(bootstrapSchema).readValues(file);
        return readValues.readAll();
    } catch (Exception e) {
        logger.error("Error occurred while loading object list from file " + fileName, e);
        return Collections.emptyList();
    }
}
```

注意事项:

*   我们基于第一个“标题”行创建了`CSVSchema`。
*   该实现足够通用，可以处理任何类型的对象。
*   如果出现任何错误，将返回一个空列表。

### 3.2。处理多对多关系

Jackson CSV 不太支持嵌套对象——我们需要使用间接方式加载多对多关系。

我们将把这些**表示为类似于简单的连接表**——因此自然地，我们将从磁盘加载数组列表:

```java
public List<String[]> loadManyToManyRelationship(String fileName) {
    try {
        CsvMapper mapper = new CsvMapper();
        CsvSchema bootstrapSchema = CsvSchema.emptySchema().withSkipFirstDataRow(true);
        mapper.enable(CsvParser.Feature.WRAP_AS_ARRAY);
        File file = new ClassPathResource(fileName).getFile();
        MappingIterator<String[]> readValues = 
          mapper.reader(String[].class).with(bootstrapSchema).readValues(file);
        return readValues.readAll();
    } catch (Exception e) {
        logger.error(
          "Error occurred while loading many to many relationship from file = " + fileName, e);
        return Collections.emptyList();
    }
}
```

下面是如何在一个简单的 CSV 文件中表示这些关系之一—`Roles <-> Privileges`:

```java
role,privilege
ROLE_ADMIN,ADMIN_READ_PRIVILEGE
ROLE_ADMIN,ADMIN_WRITE_PRIVILEGE
ROLE_SUPER_USER,POST_UNLIMITED_PRIVILEGE
ROLE_USER,POST_LIMITED_PRIVILEGE
```

注意我们在这个实现中是如何忽略头部的，因为我们并不真正需要那个信息。

## 4。设置数据

现在，我们将使用一个简单的`Setup` bean 来完成从 CSV 文件设置权限、角色和用户的所有工作:

```java
@Component
public class Setup {
    ...

    @PostConstruct
    private void setupData() {
        setupRolesAndPrivileges();
        setupUsers();
    }

    ...
}
```

### 4.1。设置角色和权限

首先，让我们将**角色和权限**从磁盘加载到工作内存中，然后将它们作为设置过程的一部分保存下来:

```java
public List<Privilege> getPrivileges() {
    return csvDataLoader.loadObjectList(Privilege.class, PRIVILEGES_FILE);
}

public List<Role> getRoles() {
    List<Privilege> allPrivileges = getPrivileges();
    List<Role> roles = csvDataLoader.loadObjectList(Role.class, ROLES_FILE);
    List<String[]> rolesPrivileges = csvDataLoader.
      loadManyToManyRelationship(SetupData.ROLES_PRIVILEGES_FILE);

    for (String[] rolePrivilege : rolesPrivileges) {
        Role role = findRoleByName(roles, rolePrivilege[0]);
        Set<Privilege> privileges = role.getPrivileges();
        if (privileges == null) {
            privileges = new HashSet<Privilege>();
        }
        privileges.add(findPrivilegeByName(allPrivileges, rolePrivilege[1]));
        role.setPrivileges(privileges);
    }
    return roles;
}

private Role findRoleByName(List<Role> roles, String roleName) {
    return roles.stream().
      filter(item -> item.getName().equals(roleName)).findFirst().get();
}

private Privilege findPrivilegeByName(List<Privilege> allPrivileges, String privilegeName) {
    return allPrivileges.stream().
      filter(item -> item.getName().equals(privilegeName)).findFirst().get();
}
```

然后我们将在这里做持久化工作:

```java
private void setupRolesAndPrivileges() {
    List<Privilege> privileges = setupData.getPrivileges();
    for (Privilege privilege : privileges) {
        setupService.setupPrivilege(privilege);
    }

    List<Role> roles = setupData.getRoles();
    for (Role role : roles) {
        setupService.setupRole(role);
    }
}
```

这是我们的`SetupService`:

```java
public void setupPrivilege(Privilege privilege) {
    if (privilegeRepository.findByName(privilege.getName()) == null) {
        privilegeRepository.save(privilege);
    }
}

public void setupRole(Role role) {
    if (roleRepository.findByName(role.getName()) == null) { 
        Set<Privilege> privileges = role.getPrivileges(); 
        Set<Privilege> persistedPrivileges = new HashSet<Privilege>();
        for (Privilege privilege : privileges) { 
            persistedPrivileges.add(privilegeRepository.findByName(privilege.getName())); 
        } 
        role.setPrivileges(persistedPrivileges); 
        roleRepository.save(role); }
}
```

请注意，在我们将角色和特权加载到工作内存中之后，我们是如何一个接一个地加载它们的关系的。

### 4.2。设置初始用户

接下来——让我们将**个用户**加载到内存中，并保存他们:

```java
public List<User> getUsers() {
    List<Role> allRoles = getRoles();
    List<User> users = csvDataLoader.loadObjectList(User.class, SetupData.USERS_FILE);
    List<String[]> usersRoles = csvDataLoader.
      loadManyToManyRelationship(SetupData.USERS_ROLES_FILE);

    for (String[] userRole : usersRoles) {
        User user = findByUserByUsername(users, userRole[0]);
        Set<Role> roles = user.getRoles();
        if (roles == null) {
            roles = new HashSet<Role>();
        }
        roles.add(findRoleByName(allRoles, userRole[1]));
        user.setRoles(roles);
    }
    return users;
}

private User findByUserByUsername(List<User> users, String username) {
    return users.stream().
      filter(item -> item.getUsername().equals(username)).findFirst().get();
}
```

接下来，让我们关注持久化用户:

```java
private void setupUsers() {
    List<User> users = setupData.getUsers();
    for (User user : users) {
        setupService.setupUser(user);
    }
}
```

这是我们的`SetupService`:

```java
@Transactional
public void setupUser(User user) {
    try {
        setupUserInternal(user);
    } catch (Exception e) {
        logger.error("Error occurred while saving user " + user.toString(), e);
    }
}

private void setupUserInternal(User user) {
    if (userRepository.findByUsername(user.getUsername()) == null) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        user.setPreference(createSimplePreference(user));
        Set<Role> roles = user.getRoles(); 
        Set<Role> persistedRoles = new HashSet<Role>(); 
        for (Role role : roles) { 
            persistedRoles.add(roleRepository.findByName(role.getName())); 
        } 
        user.setRoles(persistedRoles);
        userRepository.save(user);
    }
}
```

这里是`createSimplePreference()`方法:

```java
private Preference createSimplePreference(User user) {
    Preference pref = new Preference();
    pref.setId(user.getId());
    pref.setTimezone(TimeZone.getDefault().getID());
    pref.setEmail(user.getUsername() + "@test.com");
    return preferenceRepository.save(pref);
}
```

请注意，在我们保存用户之前，我们是如何为它创建一个简单的`Preference`实体并首先持久化它的。

## 5。测试 CSV 数据加载器

接下来，让我们对我们的`CsvDataLoader`执行一个简单的单元测试:

我们将测试用户、角色和权限列表的加载:

```java
@Test
public void whenLoadingUsersFromCsvFile_thenLoaded() {
    List<User> users = csvDataLoader.
      loadObjectList(User.class, CsvDataLoader.USERS_FILE);
    assertFalse(users.isEmpty());
}

@Test
public void whenLoadingRolesFromCsvFile_thenLoaded() {
    List<Role> roles = csvDataLoader.
      loadObjectList(Role.class, CsvDataLoader.ROLES_FILE);
    assertFalse(roles.isEmpty());
}

@Test
public void whenLoadingPrivilegesFromCsvFile_thenLoaded() {
    List<Privilege> privileges = csvDataLoader.
      loadObjectList(Privilege.class, CsvDataLoader.PRIVILEGES_FILE);
    assertFalse(privileges.isEmpty());
}
```

接下来，让我们通过数据加载器测试加载一些多对多关系:

```java
@Test
public void whenLoadingUsersRolesRelationFromCsvFile_thenLoaded() {
    List<String[]> usersRoles = csvDataLoader.
      loadManyToManyRelationship(CsvDataLoader.USERS_ROLES_FILE);
    assertFalse(usersRoles.isEmpty());
}

@Test
public void whenLoadingRolesPrivilegesRelationFromCsvFile_thenLoaded() {
    List<String[]> rolesPrivileges = csvDataLoader.
      loadManyToManyRelationship(CsvDataLoader.ROLES_PRIVILEGES_FILE);
    assertFalse(rolesPrivileges.isEmpty());
}
```

## 6。测试设置数据

最后，让我们对 bean `SetupData`执行一个简单的单元测试:

```java
@Test
public void whenGettingUsersFromCsvFile_thenCorrect() {
    List<User> users = setupData.getUsers();

    assertFalse(users.isEmpty());
    for (User user : users) {
        assertFalse(user.getRoles().isEmpty());
    }
}

@Test
public void whenGettingRolesFromCsvFile_thenCorrect() {
    List<Role> roles = setupData.getRoles();

    assertFalse(roles.isEmpty());
    for (Role role : roles) {
        assertFalse(role.getPrivileges().isEmpty());
    }
}

@Test
public void whenGettingPrivilegesFromCsvFile_thenCorrect() {
    List<Privilege> privileges = setupData.getPrivileges();
    assertFalse(privileges.isEmpty());
}
```

## 7。结论

在这篇简短的文章中，我们探讨了通常需要在启动时加载到系统中的初始数据的另一种设置方法。当然，这只是一个简单的概念证明和一个良好的基础—**而不是一个生产就绪的解决方案**。

我们还将在正在进行的案例研究所跟踪的 Reddit web 应用程序中使用该解决方案。