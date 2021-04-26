---
layout: post
title: "从零开始实现前端表单验证插件"
date: 2017-03-11 08:00:00 +0800
comments: true
categories: frontend
---

<!-- more -->

> 本文使用 es6、不依赖任何第三方库实现一个简单的表单验证插件 [github](https://github.com/Away0x/baidu-ife/blob/master/yaoyao/lesson_2/index.html)

# 效果
```html
<div class="form_group tel">
  <label for="name">手机</label>
  <input type="tel" name="tel" id="tel">
  <p class="message"></p>
</div>
```
```javascript
// 配置
{
  target: '#tel', // 验证目标
  message: { target: '.tel p', success: '手机格式正确' }, // 验证信息显示的地方
  validators: [ // 验证规则数组
    { name: 'tel', args: { message: '手机格式错误' } }
  ]
}
```

# 代码结构
- 验证插件分为 `验证类Validator`，和 `验证规则validators`
    - `验证类Validator`: 为主逻辑，负责接受表单的配置并对表单执行相应的验证规则。
    - `验证规则validators`: 其内封装了大量的验证规则，供表单选用。

每个验证规则有着相同的结构，它们接收 表单的 value 返回验证结果和报错信息。例子如下:

```javascript
// 针对手机号的验证规则
const tel = (str='', {message=''}={}) =>
({
  status: /^1[3|4|5|7|8][0-9]{9}$/.test(str), // 验证状态
  message // 报错信息
})
```

## 验证类 Validator
```javascript
class Validator {
  // 接受配置 validatorConfig: 验证器配置, targetConfig: 针对于表单的配置
  /*
  * validatorConfig {}
  *   - error   每个表单验证失败时的函数   Function
  *   - success 每个表单验证成功时的函数   Function
  *   - reset   对表单执行重置操作时的函数 Function
  * targetConfig [{}, {}, ...] -> 各个表单的配置
  *   - target        验证目标的选择器  String(Selector)
  *   - message       验证信息的配置  Object
  *     - target      验证信息显示位置的选择器  String(Selector)
  * 	- placeholder 验证信息的默认值，重置时会显示该文本 String
  * 	- success     验证成功时会显示该文本   String
  * 	- validators    验证规则的配置  [{}, {}, ...] -> 各个验证规则的配置
  * 	  - name        验证规则的名字 (validators[key])  String
  * 	  - args        验证规则的配置参数  Object
  * 	    - message   必有 该规则验证失败的显示文本  String
  * 	    - ...   	该验证规则的其他配置参数  Any (各验证规则不同)
  */
  constructor (config) {}
  // 验证主逻辑
  _check () {}
  // 单个表单的验证 (只对单个表单执行 _check)
  run () {}
  // 执行全部表单的验证 (对所有要验证的表单执行 _check)
  runAll () {}
  // 重置表单为初始状态
  reset () {}
}
```
### 验证主逻辑 _check
其接受 验证表单的配置，并执行相应验证逻辑
如接受了以下配置:

```javascript
{
  target:  '#tel', // 验证目标
  // 验证信息显示的地方
  message:  { target: '.tel p', success: '手机格式正确' }, 
  validators: [ // 验证规则数组
    { name: 'tel', args: { message: '手机格式错误' } }
  ]
}
```
需实现功能如下:

1. 对目标表单(target)执行配置中指定的验证规则 (validators: 此次配置中只有一个验证规则)
2. 如验证规则中有 required ，则说明该表单为必填项，必须进行表单验证。
3. 如验证规则中无 required ，则说明该表单为选填项，只在有输入时才进行表单验证。(该例配置为选填)
4. 当验证成功时需在 message.target 处显示 message.success 的文本。
5. 当验证失败时需在 message.target 处显示 验证规则中的指定 的文本。
6. 失败文本显示优先级同验证规则配置的先后顺序。

以下是具体实现:

```javascript
_check (config) {
    const
      validatorCfg  = this._validatorConfig,    // 验证器配置
      $target       = $(config.target),         // 验证的表单
      val           = $target.value,            // 验证的 value
      $message      = $(config.message.target), // 验证信息显示的地方
      ownValidators = config.validators,        // 所有验证规则
      result        = []                        // 失败信息数组

    let required = false // 不是必填项的话，只在有输入内容时进行检查 (默认为不必填)

    // 遍历执行验证器
    ownValidators.forEach(validator => {
      if (validator.name == 'required') required = true // 必填

      const temp = Validators[validator.name](val, validator.args) // 执行验证器

      // 如验证失败，则存储失败信息
      if ( ! temp.status) result.push( temp.message )
    })

    if ( ! required && val.length === 0) return // 可跳过检验

    if (result.length > 0) { // 有错误信息
      validatorCfg.error && validatorCfg.error($target) // 失败回调
      $message.textContent = result[0]
    }
    else { // 验证通过
      validatorCfg.success && validatorCfg.success($target) // 成功回调
      $message.textContent = config.message.success || ''
    }
  }
```
```javascript
// 单个验证 需传入要验证 input 的 id
run (target) {
  // 得到当前要验证的表单的配置
  const curConfig = this._targetConfig.filter(config => config.target === `#${target}`)[0]

  this._check(curConfig)
}
// 执行全局验证
runAll () {
  this._targetConfig.forEach(this._check.bind(this))
}
// 重置表单
reset () {
  this._targetConfig.forEach(config => {
    const
      $target  = $(config.target),
      $message = $(config.message.target)

    $target.value        = ''
    $message.textContent = config.message.placeholder || ''
  })
  this._validatorConfig.reset( this._allTarget ) // 重置回调
}
```
自此我们的验证插件就写完了，可自行在 validators 中编写更多的验证规则

# 继续优化
* 提交表单时应需得到所有表单的验证状态，全通过则提交。
* 目前失败信息提示过于单一。
* 如表单验证中需与后台通信(如验证用户名是否注册),那么验证器应该支持异步的验证规则。
* 目前会执行所有验证规则，可选择在验证规则失败时，就不再往下继续验证了，以提高性能。