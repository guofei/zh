---
layout: post
title: "ios addressbook frameword使用方法"
date: 2012-05-05 18:41
comments: true
categories: [objective-c,ios,addressbook]
---
基本上是苹果的这篇文章的翻译
[address book framework](https://developer.apple.com/library/ios/#documentation/ContactData/Conceptual/AddressBookProgrammingGuideforiPhone/Chapters/BasicObjects.html#//apple_ref/doc/uid/TP40007744-CH3-SW1)
### address books, records, single-value properties, multivalue propertie
操作address books的数据库需要理解4个object：address books, records, single-value properties, and multivalue propertie

### Address Book object
可以通过ABAddressBookRef来声明AddressBook object，然后通过ABAddressBookCreate来初始化这个object，对这个object进行操作之后，可以通过ABAddressBookSave来保存。通过ABAddressBookRevert来放弃保存，ABAddressBookHasUnsavedChanges来确认是否有没保存的变更

简单的例子
{% highlight objective-c %}
ABAddressBookRef addressBook;
bool wantToSaveChanges = YES;
bool didSave;
CFErrorRef error = NULL;
addressBook = ABAddressBookCreate();
/* ... Work with the address book. ... */

if (ABAddressBookHasUnsavedChanges(addressBook)) {
    if (wantToSaveChanges) {
         didSave = ABAddressBookSave(addressBook, &error);
         if (!didSave) {/* Handle error here. */}
     } else {
         ABAddressBookRevert(addressBook);
     }
}

CFRelease(addressBook);
{% endhighlight %}

### Records
Records通常作为AddressBook的一部分，通过ABRecordRef来声明，一个Record表示一个人或者一个group，通过ABRecordGetRecordType函数可以取得record的type，如果是一个人的话则返回kABPersonType，是一个group的话则返回kABGroupType，不管是person对象还是group对象，他们都是同一个class的instance。

person record有电话，姓名等属性，group record则有多个联系人的属性，ABGroupCopyArrayOfAllMembers可以获取所以联系人，

操作record的函数有ABRecordCopyValue，ABRecordSetValue和ABRecordRemoveValue函数。

### Properties
单一值属性

比如联系人姓名，只有一个属性

获取单一值属性，并赋值
{% highlight objective-c %}
ABRecordRef aRecord = ABPersonCreate();
CFErrorRef anError = NULL;
bool didSet;

didSet = ABRecordSetValue(aRecord, kABPersonFirstNameProperty, CFSTR("Katie"), &anError);
if (!didSet) {/* Handle error here. */}

didSet = ABRecordSetValue(aRecord, kABPersonLastNameProperty, CFSTR("Bell"), &anError);
if (!didSet) {/* Handle error here. */}

CFStringRef firstName, lastName;
firstName = ABRecordCopyValue(aRecord, kABPersonFirstNameProperty);
lastName  = ABRecordCopyValue(aRecord, kABPersonLastNameProperty);

/* ... Do something with firstName and lastName. ... */

CFRelease(aRecord);
CFRelease(firstName);
{% endhighlight %}

多值属性

联系人的电话号码可以有多个，因此是个多值属性，多值属性是一个list，list中的每个值都由Label，value，ID构成


### 用户对addressbook的选取和表示
* ABPeoplePickerNavigationController：选取person record
* ABPersonViewController:显示用户的person record
* ABNewPersonViewController：生成新的person record
* ABUnknownPersonViewController：让用户完成未完成的person record

### 程序直接访问addressbook
* record的增加和删除：ABAddressBookAddRecord,ABAddressBookRemoveRecord
* record的搜索：ABAddressBookCopyPeopleWithName，ABAddressBookGetPersonWithRecordID
* 获取所有record：ABAddressBookCopyArrayOfAllPeople

对所有record进行排序
{% highlight objective-c %}
ABAddressBookRef addressBook = ABAddressBookCreate();
CFArrayRef people = ABAddressBookCopyArrayOfAllPeople(addressBook);
CFMutableArrayRef peopleMutable = CFArrayCreateMutableCopy(
                                          kCFAllocatorDefault,
										  CFArrayGetCount(people),
										  people
                                  );
CFArraySortValues(
        peopleMutable,
        CFRangeMake(0, CFArrayGetCount(peopleMutable)),
        (CFComparatorFunction) ABPersonComparePeopleByName,
        (void*) ABPersonGetSortOrdering()
);
CFRelease(addressBook);
CFRelease(people);
CFRelease(peopleMutable);
{% endhighlight %}

获取联系人信息
{% highlight objective-c %}
ABAddressBookRef book = ABAddressBookCreate();
// Count of Addressbook
//CFIndex cnt = ABAddressBookGetPersonCount(book);
//AllRecords of Addressbook
CFArrayRef records = ABAddressBookCopyArrayOfAllPeople(book);
for (int i = 0; i < CFArrayGetCount(records); i++) {
    ABRecordRef person = CFArrayGetValueAtIndex(records, i);
    CFDateRef cfdate = ABRecordCopyValue(person, kABPersonModificationDateProperty);

    //获取姓名
    NSString *firstName = (NSString *)ABRecordCopyValue(person, kABPersonFirstNameProperty);
    NSString *lastName = (NSString *)ABRecordCopyValue(person, kABPersonLastNameProperty);
    NSString *name = [NSString stringWithFormat:@"%@ %@",lastName, firstName];
    CFRelease(firstName);
    CFRelease(lastName);

    //获取便签
    NSString *memo = (NSString *)ABRecordCopyValue(person, kABPersonNoteProperty);

    //电话
    NSString *tel1, *tel2;
    ABMultiValueRef tels = ABRecordCopyValue(person, kABPersonPhoneProperty);
    if (ABMultiValueGetCount(tels) > 0) {
        tel1 = (NSString *)ABMultiValueCopyValueAtIndex(tels, 0);
        if (ABMultiValueGetCount(tels) > 1) {
            tel2 = (NSString *)ABMultiValueCopyValueAtIndex(tels, 1);
        }
    }
    CFRelease(tels);

    //Email
    ABMultiValueRef emails = ABRecordCopyValue(person, kABPersonEmailProperty);
    NSString *email;
    if (ABMultiValueGetCount(emails) > 0) {
        email = (NSString *)ABMultiValueCopyValueAtIndex(emails, 0);
    }
    CFRelease(emails);

    //URL
    ABMultiValueRef urls = ABRecordCopyValue(person, kABPersonURLProperty);
    NSString *url;
    if (ABMultiValueGetCount(urls) > 0) {
        url = (NSString *)ABMultiValueCopyValueAtIndex(urls, 0);
    }
    CFRelease(urls);

    //地址
    ABMultiValueRef addresses = ABRecordCopyValue(person, kABPersonAddressProperty);
    NSString *address;
    if (ABMultiValueGetCount(addresses) > 0) {
        CFDictionaryRef dict = ABMultiValueCopyValueAtIndex(addresses, 0);
        CFStringRef street = CFDictionaryGetValue(dict, kABPersonAddressStreetKey);
        CFStringRef city = CFDictionaryGetValue(dict, kABPersonAddressCityKey);
        //CFStringRef country = CFDictionaryGetValue(dict, kABPersonAddressCountryKey);
        CFRelease(dict);

        // address = [NSString stringWithFormat:@"%@, %@, %@",street,city,country];
        address = [NSString stringWithFormat:@"%@, %@",street,city];
    }
    CFRelease(addresses);

    CFRelease(address);
    CFRelease(url);
    CFRelease(email);
    CFRelease(tel2);
    CFRelease(tel1);
    CFRelease(memo);
}
CFRelease(records);
CFRelease(book);
{% endhighlight %}
