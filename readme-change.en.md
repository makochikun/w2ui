# Notes to myself about changes and additions

First of all, I would like to express my respect and gratitude to vitmalina for creating this wonderful library.

I'm Japanese and not good at English, and to be honest, I don't really understand how to use GitHub. Please let me know if I caused any trouble to the creator, vitmalina, or others. To immediately delete.

## Changes made to w2grid

### (1) Changed so that functions can be set in the url option.

I wanted to get data from the server via socket.io in the main process of Electron and reflect the data received in the render process in the grid, so I wanted to set a function in the url, so I decided to change it.

The POST data originally sent to the server by Fetch is passed as an object to the function set in the url.

I'm not good at explaining it in words, so I'll try to write a rough source sample.



```js
let grid = w2grid({
   url: dataSource,
   postData: [
     cmd: 'test'
   ],
   sortData: [
     { field: 'firstName', direction: 'asc' },
     { field: 'lastName', direction: 'asc' }
   ],
   columns: [
     { field: 'firstName', type: text, sortable: true },
     { field: 'lastName', type: text }
   ]
})

function dataSource(request) {
   console.log(request)
   return new Promise(async (res,rej) => {
     server( request ).then((result)=>{
       console.log(result)
       res({status: 'success', data: result})
     }).catch((result)=>{
       rej({status:'error', statusText: result})
     })
   })
}

---
{
   cmd: 'test',
   sort: {
     {field: 'firstName', direction: 'asc'},
     {field: 'lastName', direction: 'asc'}
   },
   limit: 100,
   offset: 0
}

{
   status: 'success',
   total: 655,
   records: [
     { firstName:'Nakamura', lastName:'Masaaki' },
     { firstName:'Nakamura', lastName:'Yuki' },
     { firstName:'Minase', lastName:'Naoto' }
   ]
}
```

Please refer to the example above.

- (2023-06-27) At this point, I've only tested up to the point of getting data. We will gradually test and resolve saving and deleting.

---

## Changes made to w2form

### (1) You can now set styles for columns

```js
'Tab title': {
   type: 'tab',
   fields: {
     'Group title1': {
       type: 'group',
       titleStyle: 'font-size: 1em; font-weight: bold;',
       style: 'width: 100%',
       columns: 0,
       columnStyle: 'width: 80%',
       fields: {
         'request': { type: 'textarea', required: false, label: '', span: 0 }
       }
     },
     'Group title2': {
       type: 'group', span: 0,
       titleStyle: 'font-size: 1em; font-weight: bold;',
       columns: 1,
       columnStyle: 'width: 20%; min-width: 150px;',
       fields: {
         'request': { type: 'textarea', required: false, label: '', span: 0 }
       }
     }
   }
```
### (2) Added view mode of form

There is a method to make each field read-only individually, so it would be nice to use it, but it's troublesome, so I added it.

```js
const form = new w2form({
   name: 'mainForm',
   header: 'Header',
   readOnly: true, // true or false
   fields: {
   }

----

form.formReadOnly(false) // true = change to read only mode
                          // false = change to editable mode

```

### (3) Added reset method

I added it because I often want to use it
A method that simply refresh() after copying form.original to form.record using Object.assign


```js
form.reset()
```
---

## Changes made to w2tabs

### (1) Added a function to attach badges to tabs

```js
const tabs = new w2tabs({
   active: 'all',
   style: 'padding-top: 10px',
   tabs: [
     { id: 'tab1',
       text: 'all'
     },
     { id: 'tab2',
       closable: true,
       text: 'prepare'
     },
     { id: 'tab3',
       text: 'waiting',
       badge: { // Only set the default badge settings for the tabs you want to change
         counter: 0, // number to display on the badge
         bgColor: 'blue', // background color of the badge
         time: '0.5s', // Flashing time per time
         blink: '3' // number of blinks
       }
     }
   ],
})

tabs.setBadge('tab1', {counter:2, bgColor:'pink', blink:5})
tabs.setBadge('tab2', 3)
tabs.setBadge('tab3', 1)
```

#### setBadge( [tab id], [number] or [object])

Badges can only be set numerically and have a maximum display limit of 99. Even if you set a number greater than 99, only 99 will be displayed.

Setting it to 0 will make the badge disappear.

For blink, you can specify the number of times with a numerical value or set 'infinite' etc.

With the addition of the badge function, the position of the Close button has been changed from the right side of the tab to the left side.
