# 1. 什么是 DDD ?
 DDD(Domain-Driven Design 领域驱动设计), 可以认为是一种设计模式,    
 不过 DDD 主要解决如果让代码更加“业务”一些。  
 - 业务需求在代码中实现后，如果不加任何注释，如何让后期的维护人员能够一目了然地知道业务需求？  
 - 如何让程序员和产品、以及其他项目组的业务人员使用同一种沟通语言，提高沟通效率？  

# 2. A Glance of Master's Work
通过一个例子，看看大神是怎么做的，从而来体会体会一下到底怎么在代码中实现 DDD。  

## 2.1 Code style we can see here and there
我们会经常这么写代码，或者经常看见这样的代码  
```
@Transactional
public void saveCustomer(
    String customerId,
    String customerFirstName, String customerLastName,
    String streetAddress1, String streetAddress2,
    String city, String stateOrProvince,
    String postalCode, String country,
    String homePhone, String mobilePhone,
    String primaryEmailAddress, String secondaryEmailAddress) {
    Customer customer = customerDao.readCustomer(customerId);
    if (customer == null) {
        customer = new Customer();
        customer.setCustomerId(customerId);
    }
    customer.setCustomerFirstName(customerFirstName);
    customer.setCustomerLastName(customerLastName);
    customer.setStreetAddress1(streetAddress1);
    customer.setStreetAddress2(streetAddress2);
    customer.setCity(city);
    customer.setStateOrProvince(stateOrProvince);
    customer.setPostalCode(postalCode);
    customer.setCountry(country);
    customer.setHomePhone(homePhone);
    customer.setMobilePhone(mobilePhone);
    customer.setPrimaryEmailAddress(primaryEmailAddress);
    customer.setSecondaryEmailAddress (secondaryEmailAddress);
    customerDao.saveCustomer(customer);
}

```
这种写法很通用，一个方法可以被各种业务调用，达到了 write once, used anywhere   
然而，过了 3 个月之后，自己回过头来看，其实已经不知道当初的业务逻辑是什么了   
这时候该让 DDD 出场了。。。

## 2.2  Master's tactical code with DDD  

大师会这么写  
```
public interface Customer {
    public void changePersonalName(String firstName, String lastName);
    public void postalAddress(PostalAddress postalAddress);
    public void relocateTo(PostalAddress changedPostalAddress);
    public void changeHomeTelephone(Telephone telephone);
    public void disconnectHomeTelephone();
    public void changeMobileTelephone(Telephone telephone);
    public void disconnectMobileTelephone();
    public void primaryEmailAddress(EmailAddress emailAddress);
    public void secondaryEmailAddress(EmailAddress emailAddress);
}

```
然后再去实现这个接口。  
是不是感觉这段代码更接近产品、业务逻辑的需要了？  
或者说读起来感觉就知道要解决的业务需求了？   
这就是 DDD 要解决的实际问题。   

# 3. Reference

[1] Vaughn Vernon, Implementing Domain-Driven Design (1st ed.), 978-0321834577, Addison-Wesley Professional, Feb 16, 2013.
