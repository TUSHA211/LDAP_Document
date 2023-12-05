# **LDAP**
## **Lightweight   Directory  Access  Protocol**


 

## Table of Content
********************
  
* [Environment Info](#Environment-Info)

* [Overview of LDAP](#Overview-of-LDAP) 

* [Installation of Podman](#Installation-of-Podman)   

* [Key Features of Ldap](#Key-Features-of-Ldap)     

* [Installation of Podman](#Installation-of-Podman)  

* [Install Ldap utility on bash machine](#Install-Ldap-utility-on-bash-machine)  

* [Apache Directory Studio](#Apache-Directory-Studio)




#### Environment Info 
**Server Info-**  
Os version  
NAME="Ubuntu"  
VERSION="20.04.6 LTS (Focal Fossa)"  
podman version 3.4.2  


**Client Info-**  
NAME="Ubuntu"  
VERSION="20.04.6 LTS (Focal Fossa)"  
Ldap version:- 3  
Apache Directory Studio version:- 2.0.0.v20210717-M17

## List of Tools:- 

* Podman
* Ldap
* Apache Directory Studio


### Overview of LDAP

**LDAP**  - The Lightweight Directory Access Protocol is a communication protocol used to access directory servers and is used to store, update and retrieve data from a directory structure.

#### key concepts related to LDAP:

**Directory Services:** LDAP is commonly used for directory services, which organize and provide access to information in a hierarchical, tree-like structure. This structure is often used to represent an organization's organizational units, users, groups, and other objects.

**Directory Information Tree (DIT):** The data in an LDAP directory is organized into a hierarchy called the Directory Information Tree (DIT). The DIT is a tree-like structure where each node represents an object, and the relationships between nodes reflect the organizational structure.

**Entries:**  An entry is a fundamental unit of data in LDAP, representing an object in the directory. Each entry has a unique Distinguished Name (DN) that identifies its position in the DIT.

**Attributes:** Entries contain attributes that store information about the object. Attributes can have one or more values and define characteristics such as names, addresses, and access control information.

**Distinguished Name (DN):** A DN uniquely identifies an entry in the LDAP directory. It consists of the entry's name and its position in the DIT. The DN provides a way to locate and reference a specific entry.


### Podman

Podman is a container management tool that is used to manage pods, containers, and container images on a Linux system.Podman is part of the container ecosystem, and it is designed to be daemonless and lightweight.

#### Installation of Podman

```
sudo apt install podman
```
```
Output-
Reading package lists... Done
Building dependency tree  	 
Reading state information... Done
podman is already the newest version (100:3.4.2-5).
``````


 
### Command for the setup or configuration 
#### 1.Create Pod and container  
```
 vim ldap.sh
```
* If vim not install-
  
```
sudo apt install vim
```
- **Create pod**
```   
podman pod create --name ldap389 --publish 3389:3389
```
 - **Create container**

- Change path **/home/tusha**
```
pwd
```  
```
#!/bin/bash
podman run -it  --pod ldap389 --name 389ds-ldap -v /home/tusha:/data -e DS_SUFFIX_NAME=dc=keen,dc=in -e DS_DM_PASSWORD=1 docker.io/389ds/dirsrv
```
* After paste this command in the container save and quit
```
:wq
```
  
* After run this container, then exit
  
  ```
  Ctrl+Q
  ```
  

* In Pod we have set port 3389
* In container we have set base Dn as keen.in and set password of 1

    - Check suffix created or not
    - Go to inside container
  ``` 
    podman exec -it  389ds-ldap bash
  ```

 * Create new Suffix
  ```
  dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend suffix list
  ```
 ```
 Output-
 ldap389:/ # dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend create --suffix="dc=keen,dc=in" --be-name="keen"
--create-entries --create-suffix
Enter password for cn=Directory Manager on ldap://localhost:3389:
The database was sucessfully created
ldap389:/ # dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend create --suffix="dc=keen,dc=in" --be-name="keen" --create-entries --create-suffix
Enter password for cn=Directory Manager on ldap://localhost:3389:
Error: Mapping tree for this suffix exists!
ldap389:/ # dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend suffix list
Enter password for cn=Directory Manager on ldap://localhost:3389:
dc=keen,dc=in (keen)
```

 *  If suffix have not created then run this command
 ```
 dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend create --suffix="dc=keen,dc=in" --be-name="keen"
 --create-entries --create-suffix
 ```
``` 
exit
```
```
Output-
ldap389:/ # exit
exit
```

  
#### Install Ldap utility on bash machine

  a. Check container list
```
 podman ps -a 
```
b. Install Ldap utility on bash machine  

 ``` 
 sudo apt install ldap-utils
  ```
```
Output-
Reading package lists... Done
Building dependency tree  	 
Reading state information... Done
ldap-utils is already the newest version (2.4.49+dfsg-2ubuntu1.9).
```

#### 2.Create Organization_unit ldif file (file extension name,ldif ) 

- we have create 5 organisation ( Dev, Support,SupportTm, Document, Observation)
```
vim organisation.ldif
```
```
dn: dc=keen,dc=in
objectClass: top
objectClass: domain
objectClass: organizationalUnit
ou:  keen
dc: keen

dn: ou=Dev,dc=keen,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Dev

dn: ou=Support,dc=keen,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Support

dn: ou=SupportTm,dc=keen,dc=in
objectClass: top
objectClass: organizationalUnit
ou: SupportTm

dn: ou=Document,dc=keen,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Document

dn: ou=Observation,dc=keen,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Observation
````

- Run this command to add organisation.ldif

``` 	
ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W -f organisation.ldif
```
```
Output-
tusha@tusha:~$ ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory   Manager" -W  -f organisation.ldif
Enter LDAP Password:
adding new entry "dc=keen,dc=in"
adding new entry "ou=Dev,dc=keen,dc=in"
adding new entry "ou=Support,dc=keen,dc=in"
adding new entry "ou=SupportTm,dc=keen,dc=in"
adding new entry "ou=Document,dc=keen,dc=in"
adding new entry "ou=Observation,dc=keen,dc=in"
```


### 3.Create 2 group inside Support 
```
vim group.ldif
```

```
dn: cn=Admins,ou=Support,dc=keen,dc=in
objectClass: top
objectClass: groupOfUniqueNames
cn: Admins
uniqueMember: uid=user1,ou=Support,dc=keen,dc=in     

dn: cn=SupportTeam,ou=Support,dc=keen,dc=in
objectClass: top
objectClass: groupOfUniqueNames
cn: SupportTeam#
uniqueMember: uid=user2,ou=Support,dc=keen,dc=in
```

A. Run this command to add organisation.ldif 
``` 	
ldapadd  -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W  -f group.ldif
```

```
Output-
tusha@tusha:~$ ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W  -f group.ldif
Enter LDAP Password:
adding new entry "cn=Admins,ou=Support,dc=keen,dc=in"
adding new entry "cn=SupportTeam,ou=Support,dc=keen,dc=in"
``````



#### 4.Create Custom attribute according to our requirement

- Create customer attribute ldif file
```
vim custom_attribute.ldif
```

```
dn: cn=schema
changetype: modify
add: attributeTypes
attributetypes: (emp_code-oid NAME  'EmployeeCode' DESC 'EmployeeCode' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} SINGLE-VALUE X-ORIGIN 'user defined' )
attributetypes: (gender-oid NAME 'Gender' DESC 'Gender' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (certifications-oid NAME 'Certifications' DESC 'Certifications' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} X-ORIGIN 'user defined')
attributetypes: (passport-oid NAME 'PassportNo'  DESC 'PassportNo.' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (pan_no-oid NAME 'Panno' DESC 'Panno.' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (qualification-oid NAME 'Qualification' DESC 'Qualification' EQUALITY caseIgnoreMatch  SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} X-ORIGIN 'user defined')
attributetypes: (correspondence_address-oid  NAME 'CorrespondenceAddress' DESC 'CorrespondenceAddress' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{100} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (personalemail-id-oid NAME 'personalemail-id' DESC 'personalemail-id' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (facebookaccount-oid NAME 'facebookaccount' DESC 'facebookaccount' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (twitteraccount-oid NAME 'twitteraccount' DESC 'twitteraccount' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{15} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (MaritalStatus-oid NAME 'MaritalStatus' DESC 'MaritalStatus' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (Childinfo-oid NAME 'Childinfo' DESC 'Childinfo' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} X-ORIGIN 'user defined')
attributetypes: (pfno-oid NAME 'pfno' DESC 'pfno' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (bankname-oid NAME 'BankName' DESC 'BankName' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (AccountNo-oid NAME 'AccountNo' DESC 'AccountNo' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (IFSCCode-oid  NAME 'IFSCCode' DESC 'IFSCCode' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (ESICCardNo-oid NAME 'ESICCardNo' DESC 'ESICCardNo' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (FamilyMembersInsured-oid  NAME 'FamilyMembersInsured' DESC 'FamilyMembersInsured' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} X-ORIGIN 'user defined')
attributetypes: (documentssubmitted-oid  NAME 'documentssubmitted' DESC 'documentssubmitted' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{60} X-ORIGIN 'user defined')
attributetypes: (doj-oid NAME 'DateofJoining' DESC 'DateofJoining' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (doreg-oid NAME 'DateOfResignation' DESC 'DateOfResignation' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (dob-oid NAME 'DateofBirth' DESC 'DateofBirth' EQUALITY caseIgnoreIA5Match SUBSTR  caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (passport_valid_upto-oid NAME 'PassportValidupto' DESC 'PassportValidupto' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (ProfessionalStartYEARS-oid NAME 'ProfessionalStartYEARS' DESC 'ProfessionalStartYEARS' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (YEARSOfExperience-oid NAME 'YEARSOfExperience' DESC 'YEARSOfExperience' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (dateofjoiningasintern-oid NAME 'dateofjoiningasintern' DESC 'dateofjoiningasintern'  EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (AadharNo-oid  NAME 'AadhaarNo' DESC 'AadhaarNo' EQUALITY integerMatch SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{20}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (YearsofQualification NAME 'YearsofQualification' DESC 'YearsofQualification' EQUALITY integerMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (mobileno-oid  NAME 'mobileno' DESC 'mobileno' EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{20} X-ORIGIN 'user defined')
attributetypes: (UANno-oid NAME 'UANno' DESC 'UANno' EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{20}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (InsuranceMonthlyAmountDeductionINR-oid NAME 'InsuranceMonthlyAmountDeductionINR' DESC 'InsuranceMonthlyAmountDeductionINR'  EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: ( projectname-oid NAME 'ProjectName' DESC 'ProjectName' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{50} SINGLE-VALUE X-ORIGIN 'user defined')

```

B.Add this file to ldap db

```
ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W -f custom_attribute.ldif
```

```
Output-
 tusha@tusha:~$ ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W -f custom_attribute.ldif
Enter LDAP Password:
modifying entry "cn=schema"

``````



#### 5.Create Object class file for add attribute to object class
```
vim object_class.ldif
```

```
dn: cn=schema
changetype: modify
add: objectClasses
objectClasses: ( customEmployee-oid NAME 'customEmployee' SUP top STRUCTURAL MUST ( EmployeeCode $ DateofJoining $ Gender $ DateofBirth $ Panno $ Qualification $ YearsofQualification $ ProfessionalStartYEARS $ YEARSOfExperience $ CorrespondenceAddress $ personalemail-id $ mobileno $ MaritalStatus $ BankName $ AccountNo $ IFSCCode $ documentssubmitted) MAY ( DateOfResignation $ Certifications $ PassportNo $ PassportValidupto $ AadhaarNo $ facebookaccount $ twitteraccount $ Childinfo $ pfno $ UANno $ ESICCardNo $ InsuranceMonthlyAmountDeductionINR $ FamilyMembersInsured $ dateofjoiningasintern $ ProjectName )  X-ORIGIN 'user defined')

```

A.Add object class ldif file

```
ldadadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W  -f object_class.ldif
```
```
Output-
tusha@tusha:~$ ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W -f object_class.ldif
Enter LDAP Password:
modifying entry "cn=schema"

``````

#### 6.Create user with custom attribute
```
vim user1.ldif
```

```
dn: uid=001,ou=dev,dc=keen,dc=in
objectClass: top
objectClass: inetOrgPerson
objectClass: customEmployee
cn: rahul
sn: Gupta
uid: 001
EmployeeCode: 101
userPassword: 12345@
personalemail-id: rahulgupta@yopmail.com
mobileNo: 1213141500
documentssubmitted: yes
DateofJoining: 05-01-2022
Gender: Male
DateofBirth: 06-01-1998
Panno: ABCDH7654P
Qualification: MCA
YearsofQualification: 2021
ProfessionalStartYEARS: 2022
YEARSOfExperience: 01
CorrespondenceAddress: Delhi
MaritalStatus: Single
BankName: PNB
AccountNo: 1100110011
IFSCCode: KKBK0053
  
```

A.In this file we have gave objectclass name and must custom attribute


B.After that add user to db

```
ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W -f user1.ldif
```

```
Output-
tusha@tusha:~$ vim user1.ldif
tusha@tusha:~$ ldapadd -a -c -xH ldap://localhost:3389 -D "cn=Directory Manager" -W -f user1.ldif
Enter LDAP Password:
adding new entry "uid=001,ou=dev,dc=keen,dc=in"
```


#### 7.Check user reflected or not through ldapsearch command

```
ldapsearch -x -D "cn=Directory Manager" -W -H ldap://localhost:3389 -b "ou=dev,dc=keen,dc=in" -s sub "(uid=001)"
```

#### Output look like this 
```
# extended LDIF
#
# LDAPv3
# base <ou=dev,dc=keen,dc=in> with scope subtree
# filter: (uid=001)
# requesting: ALL
#
# 001, dev, keen.in
dn: uid=001,ou=dev,dc=keen,dc=in
objectClass: top
objectClass: inetOrgPerson
objectClass: customEmployee
objectClass: organizationalPerson
objectClass: person
cn: rahul
sn: Gupta
uid: 001
EmployeeCode: 101
personalemail-id: rahulgupta@yopmail.com
mobileno: 1213141500
documentssubmitted: yes
DateofJoining: 05-01-2022
Gender: Male
DateofBirth: 06-01-1998
Panno: ABCDH7654P
Qualification: MCA
YearsofQualification: 2021
ProfessionalStartYEARS: 2022
YEARSOfExperience: 01
CorrespondenceAddress: Delhi
MaritalStatus: Single
BankName: PNB
AccountNo: 1100110011
IFSCCode: KKBK0053
userPassword:: e1BCS0RGMi1TSEE1MTJ9MTAwMDAkRS9XcitZajh3NkZsM001QUVoU3RiZkxJbWx
 VMDlSVVEkZGxUZmw1dVBKRW5NYVRsSklYaUUwZkRrKzNTU1U4d3hNMDVVRENKT2Q1VHVGOTQyZjA5
 MmoyLzZ0bVBESjlxWlJGbGZQK1dXeGNjRHZkNTJpUzRoMmc9PQ==

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
 tusha@tusha:~$ ldapsearch -x -D "cn=Directory Manager" -W -H ldap://localhost:3389 -b "ou=dev,dc=keen,dc=in" -s sub "(uid=001)"
Enter LDAP Password:
# extended LDIF
#
# LDAPv3
# base <ou=dev,dc=keen,dc=in> with scope subtree
# filter: (uid=001)
# requesting: ALL
#

# 001, dev, keen.in
dn: uid=001,ou=dev,dc=keen,dc=in
objectClass: top
objectClass: inetOrgPerson
objectClass: customEmployee
objectClass: organizationalPerson
objectClass: person
cn: rahul
sn: Gupta
uid: 001
EmployeeCode: 101
personalemail-id: rahulgupta@yopmail.com
mobileno: 1213141500
documentssubmitted: yes
DateofJoining: 05-01-2022
Gender: Male
DateofBirth: 06-01-1998
Panno: ABCDH7654P
Qualification: MCA
YearsofQualification: 2021
ProfessionalStartYEARS: 2022
YEARSOfExperience: 01
CorrespondenceAddress: Delhi
MaritalStatus: Single
BankName: PNB
AccountNo: 1100110011
IFSCCode: KKBK0053
userPassword:: e1BCS0RGMi1TSEE1MTJ9MTAwMDAkRS9XcitZajh3NkZsM001QUVoU3RiZkxJbWx
 VMDlSVVEkZGxUZmw1dVBKRW5NYVRsSklYaUUwZkRrKzNTU1U4d3hNMDVVRENKT2Q1VHVGOTQyZjA5
 MmoyLzZ0bVBESjlxWlJGbGZQK1dXeGNjRHZkNTJpUzRoMmc9PQ==

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```



 ### Apache Directory Studio

Apache Directory Studio is an open-source, cross-platform LDAP (Lightweight Directory Access Protocol) client and directory toolset. It is part of the Apache Directory  project and provides a graphical user interface for working with LDAP directories.

#### 8. Setup ApacheDirectory studio 

Create connection apache directory studio with openldap server 

To install apache directory studio

Click link below and download file


https://directory.apache.org/studio/download/download-linux.html

```
cd Downloads/
```
**cd**: Stands for "change directory," and it is used to navigate the file system.

**Downloads/**: Specifies the directory to which you want to change. The trailing slash / is used to indicate that "Download" is a directory.
```
tar -zxvf ApacheDirectoryStudio-2.0.0.v20210717-M17-linux.gtk.x86_64.tar.gz
```
**tar**: The command-line tool for handling tarball files, which are collections of files and directories bundled together. It's often used in conjunction with compression tools like gzip.

**-z**: Specifies that the input file is compressed with gzip. -x: Stands for "extract," which tells tar to extract the contents. -v: Stands for "verbose," which displays detailed information about the extraction process. -f: Specifies the name of the archive file to be processed. ApacheDirectoryStudio-2.0.0.v20210717-M17-linux.gtk.x86_64.tar.gz: This is the name of the tarball file you are extracting.

```
sudo mv ApacheDirectoryStudio-* /opt/
```
**sudo**: Stands for "superuser do." It is used to execute the command with elevated privileges. You might need administrative rights to move files into the /opt/ directory.

**mv**: Stands for "move." This command is used to move files or directories.

**ApacheDirectoryStudio**-*: This is a wildcard pattern that matches any file or directory whose name starts with "ApacheDirectoryStudio-." The asterisk * is a wildcard character that represents any sequence of characters.

**/opt/**: This is the target directory where the files will be moved. The /opt/ directory is commonly used for installing additional software on Unix-like systems.

```
sudo ln -s /opt/ApacheDirectoryStudio*/ApacheDirectoryStudio /usr/local/bin/ApacheDirectoryStudio
```

**sudo**: Executes the following command with elevated privileges.

**ln**: Stands for "link" and is used to create links between files.

**-s**: Specifies that a symbolic link should be created. Symbolic links are references to another file or directory.

/opt/ApacheDirectoryStudio*/ApacheDirectoryStudio: The source directory or file. The wildcard * is used to match any version or timestamp in the file name. This points to the Apache Directory Studio executable.

/usr/local/bin/ApacheDirectoryStudio: The destination of the symbolic link. This is the location where the symbolic link will be created.

```
sudo apt-get install openjdk-11-jre
```
**sudo**: Stands for "superuser do." It is used to execute the command with elevated privileges. You might be prompted to enter your password.

**apt-get**: The package management command-line tool used on Debian-based systems.

**install**: The install command for apt-get is used to install new packages.

**openjdk-11-jre**: This is the name of the package you want to install. It represents the OpenJDK 11 Runtime Environment.

```
Output
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
openjdk-11-jre is already the newest version (11.0.20.1+1-0ubuntu1~22.04).
openjdk-11-jre set to manually installed.
The following packages were automatically installed and are no longer required:
buildah containernetworking-plugins golang-github-containernetworking-plugin-dnsname golang-github-cont golang-github-containers-image libflashrom1 libftdi1-2 libllvm13 libllvm14 libostree-1-1
Use 'sudo apt autoremove' to remove them.
upgraded, 0 newly installed, 0 to remove and 7 not upgraded.
```
```
cd ApacheDirectoryStudio/
```

cd ApacheDirectoryStudio/: command is used to change the current working directory to the "ApacheDirectoryStudio" directory.

```
./ApacheDirectoryStudio
```

./: The dot-slash (./) is a shorthand notation indicating the current directory.

**ApacheDirectoryStudio**: This is the name of the executable or script you are trying

Now start your apache directory studio


![](https://lh7-us.googleusercontent.com/Qj4UvFAWlrEPxCv3k5M3LfBFJZ7C0Az1JjgQjBqFIhPT8D7OLVrOyMExYNdWRaFG-1ryW9VtRaUlqbDtc_gTpgjU2uYgaSX4QUoRlZm1pYvNtl9omEU04gK4IUxV3NNE7ei8e7qgB_vuSzV5EQ_hsvo)


 * After opening of Apache directory studio




![](https://lh7-us.googleusercontent.com/2Urj505_Cg3AH_y9UaW7y7LC91GjG03BuA-xY9V2DQekff08325PgSxzhpMppwZ6bcqzAyOaU5sEZ6p7PLnA_z7RxloXotMPjIjllBdreU7985_FKU1cxGIoOm5px5JlC1sAZATMslHVPlKR2EiMy_I)


####  9. Go to new connection and enter some details-

- LDAP  -> New Connection




![](https://lh7-us.googleusercontent.com/ssric_ByyNgXumAoIkMt73J1wmEUC-TfOh721U7Wd3T95LmqyPXG9JJDv5Q28-4_KA2UL5LKMHgs7t2VHxw7SoZcyh75g6e1WteVZrqqdmjIepnqXcU0K8pX5I4FF9gFl7leIdp7sn1aDunXicS2wME)


 - Write Hostname 
 - ldap port> Press next 




![](https://lh7-us.googleusercontent.com/DPMTN0zf5LZp_yZH2qwZiMQ_XPBORl6BGC2j1DWnnb-LvFPX8JCdrOIVxJzk9QIm-kYs8Hr95Adli_9qErzpA7-hRxpr_KL75Z8RW4UfFUzXYgu5VKUfWL2z0Hl1Mh_GfCAMrhiaj4xsyhCMvIMrTiw)




- Enter Bind DN name (cn= Directory Manager) 
- Enter Bind DN password -> finish



![](https://lh7-us.googleusercontent.com/Lyx_c4QKSMQvTaLULS9fGFQ9AFYopmXQPYumlBLb5uhKiPmpIKXY60D-kPgit51SFrR22VdvIM_nqKkVYum6c1ZmGiM1d7h0Z-eBK7adWyIKGaJF439E_7u5gwNzajPDH3ZbJayOHa6636hv-BOSh84)




**Then Bind our server**









