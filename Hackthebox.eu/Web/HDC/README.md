# HDC

## Accessing the website
### Analyzing resources
In jQuery file, line 10000-ish:
```
function doProcess()
{var form=document.createElement("form");	form.setAttribute("method","post");	form.setAttribute("action","main/index.php");	form.setAttribute("target","view");	var hiddenField=document.createElement("input");	hiddenField.setAttribute("type","hidden");	hiddenField.setAttribute("name","name1");	hiddenField.setAttribute("value","TXlMaXR0bGU");	var hiddenField2=document.createElement("input");	hiddenField2.setAttribute("type","hidden");	hiddenField2.setAttribute("name","name2");	hiddenField2.setAttribute("value","cDB3bmll");	form.appendChild(hiddenField2);		form.appendChild(hiddenField);	form.appendChild(hiddenField2);	document.body.appendChild(form);			window.open('','view');	form.submit();}
```
These lines seem interesting
```
hiddenField.setAttribute("name","name1");	hiddenField.setAttribute("value","TXlMaXR0bGU");
hiddenField2.setAttribute("name","name2");	hiddenField2.setAttribute("value","cDB3bmll");
```
### Getting in
Craft POST request with name1=TXlMaXR0bGU and name2=cDB3bmll

## Sending email
Browsing through page resources (or using burp spider) it is apparent that one of the pictures is sourced from a hidden directory ‘./secret_area/‘

/main/secret_area_/mails.txt - found list of emails

Import the list through Burp Suite Intruder and start attack on an email page. Finally this payload gives the flag:
‘name1=fishroesalad@mail%2ecom&name2=1&submit=Send’
