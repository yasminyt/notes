## JSON
+ JSON 全称：JavaScript Object Notation，主要用于处理 object 类型的数据，以方便在网络中进行传输
+ **`JSON.stringfy`**：将 object 转换为 JSON
+ **`JSON.parse`**：将 JSON 转换回 object
+ JSON 数据中，字符串必须、也只能用双引号

### JSON.stringfy
+ 将 object 转换为 JSON 后，生成的 json 字符串是一个名为 JSON 编码或序列化/字符串化/编组的对象
+ JSON.stringfy 可以应用于大部分数据类型中：
  + Objects { ... }
  + Arrays [ ... ]
  + 基本数据类型：strings、numbers、boolean、null
  + 全部都会被转换成 字符串
+ JSON 是 data-only cross-language（仅限数据的跨语言规范），因此 JSON.stringfy 会跳过某些特定于 js 的对象属性，包括
  + 函数属性（方法）
  + 符号属性（symbols）
  + 值为 undefined 的属性
  ```javascript
  let user = {
    sayHi() {         // 忽略
      alert('Hello')
    },
    [Symbol('id')]: 123,  // 忽略
    something: undefined  // 忽略
  }

  JSON.stringfy(user) // "{}"（返回一个空对象的字符串）
  ```
  + 如果对象中的内容都被忽略了，则会视为空对象进行处理
+ 当遇到空对象、空数组、空串时，JSON.stringfy 依然返回对应的空的表达式
  ```javascript
  JSON.stringfy({})   // "{}"
  JSON.stringfy([])   // "[]"
  JSON.stringfy('')   // """"（中间的那对双引号是表示那个空串）
  ```
+ 对于嵌套的属性，依然是会转换成字符串
  ```javascript
  let meetup = {
    title: 'Conference',
    room: {
      number: 23,
      participants: ['john', 'ann']
    }
  }

  JSON.stringfy(meetup) 
  // "{"title":"Conference","room":{"number":23,"participants":["john","ann"]}}"
  ```
+ 如果对象中存在着循环的引用，即对象A的某个属性值为对象B，而对象B的某个属性值又为对象A，则 JSON.stringfy 会报错
  ```javascript
  let room = {
    number: 23
  }
  let meetup = {
    title: 'Conference',
    participants: ['john', 'ann']
  }

  meetup.place = room
  room.occupiedBy = meetup

  JSON.stringfy(meetup)  // Error
  ```

#### JSON.stringfy 其它用途
+ 完整的参数列表为：**`JSON.stringfy(value [, replacer, space])`**
+ `replacer`参数可以让自定义要序列化的对象的属性，那么可以解决上述所说的对象存在循环引用的情况（只要不序列化产生循环应用的属性即可）
  ```javascript
  // 解决上例中存在循环引用的问题
  JSON.stringfy(meetup, ['title', 'participants', 'place', 'number']) // 'number' 是 room 中的属性
  // "{"title": "Conference", "participants": "["john", "ann"]", "place": {"number": 23}}
  ```
  + 在上述代码中，由于没有对 room 中的 occupiedBy 属性进行序列化，所以可以对 meetup 正常进行序列化
+ `replacer` 也可以是一个匿名函数，这样就不需要写特别长的属性列表，详细内容见 [Excluding and transforming: replacer](https://javascript.info/json#excluding-and-transforming-replacer)
+ `space` 参数格式化JSON数据以多少个space进行缩进（仅仅是用于方便查看，网络传输过程中，自然没有空格是最好的）

### JSON.parse
+ **`JSON.parse(str [, reviver])`**：将 str 转换回 object
+ `reviver` 是一个可选的匿名函数 `function(key, value)`，可以在转换时用来改变 value 的值
  ```javascript
  let schedule = `{
    "meetups": [
      {"title":"Conference","date":"2017-11-30T12:00:00.000Z"},
      {"title":"Birthday","date":"2017-04-18T12:00:00.000Z"}
    ]
  }`
  schedule = JSON.parse(schedule, function(key, value) {
    if (key == 'date')
      return new Date(value)    // 可以得回原来的时间日期对象，而不是一个字符串
    return value
  })
  ```