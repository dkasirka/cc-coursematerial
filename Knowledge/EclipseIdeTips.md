# FAQ - Frequently Asked Questions regarding Eclipse IDE

## 1. Java Source Code not visible
### Problem:  
When debugging the java source code is not visible.  
### Solution:  
This problem mostly occurs in case of external libraries that do not belong to the java standard. The solution is to add the source files manually. But for that you have to have access to that files.  
### How to:  
There are basically two possibilities to attach source files (e. g. you have to download the .zip file from Github etc.):  
- In the `Package Explorer` go to the `Maven Dependencies` and search for the JAR-File you want to attach the source files to, then rightclick it and select `Properties`. In the menu choose `Java Source Attachment`. Click on `External location` and select the corresponding `.zip` file or `.jar` file.  
- When you try to debug code that has no source file attached you will get an information page stating that the source file is missing. On that page there is a button to attach them. A click on that button will again open the `Properties` window. Proceed as explained above! 

## 2. Eclipse freezes
### Problem:
Eclipse freezes (e.g. after including the approuter source files [Node modules])
### Solution:
In order to prevent that behaviour you have to exclude the corresponding `Node modules` from the validation. In other words you have to specify a resource filter.
### How to:
See [exercise 22](https://github.com/ccjavadev/cc-coursematerial/blob/master/Security/Exercise_22_DeployApplicationRouter.md) for further details.

## 3. Auto-Completion of static Imports
### Problem:
Auto-Completion of static imports (e.g. `Hamcrest` matchers) does not work.
### Solution:
You have to add a new `Type Favorite` in Eclipse.
### How to:
See [exercise 4](https://github.com/ccjavadev/cc-coursematerial/blob/master/CreateMicroservice/Exercise_4_CreateServiceTests.md) for further details.

***
<dl>
  <dd>
  <div class="footer">&copy; 2018 SAP SE</div>
  </dd>
</dl>
<hr>
