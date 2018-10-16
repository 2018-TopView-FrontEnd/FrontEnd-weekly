# 虚拟列表-Virtual List（可视区域渲染）

标签（空格分隔）： 项目

---

## 一 实现思路：
因为 DOM 元素的创建和渲染需要的时间成本很高，在大数据的情况下，完整渲染列表所需要的时间不可接受。其中一个解决思路就是在任何情况下只对「可见区域」进行渲染，可以达到极高的初次渲染性能

虚拟列表指的就是「可视区域渲染」的列表，重要的基本就是两个概念：
> * 可滚动区域：假设有 1000 条数据，每个列表项的高度是 30，那么可滚动的区域的高度就是 1000 * 30。当用户改变列表的滚动条的当前滚动值的时候，会造成可见区域的内容的变更
> * 可见区域：比如列表的高度是 300，右侧有纵向滚动条可以滚动，那么视觉可见的区域就是可见区域

实现虚拟列表就是处理滚动后可见区域的变更，其中具体步骤如下：
1. 计算当前可见区域起始数据的 `startIndex`
2. 计算当前可见区域结束数据的 `endIndex`
3. 计算当前可见区域的数据，并渲染到页面中
4. 计算 `startIndex` 对应的数据在整个列表中的偏移位置 `startOffset`，并设置到列表上

[小demo][1]
____

## 二 项目应用（赛飞奇小卡片）
之前一直遗留的问题，一直没时间去优化它，最近有点闲，就着手去处理它了

思考的问题：
1. 怎么把`可视化渲染`应用在小卡片上？
2. 小卡片和上面的列表有哪些不一样的地方？


小卡片与上述例子的区别：
1. 上述例子是每行单个数据，而小卡片的数据排列是从左至右由上至下的，一行有多个数据，需考虑 **可视区域的数据量计算问题**
2. 渲染的数组数量不一样（可以理解为上述例子是一个数组的渲染，而小卡片是多个），需考虑 **过渡状态**。

____
#### 实际代码：


`1`. 小卡片的初始计算，计算出**可视区域内**第一个小卡片和最后一个小卡片的位置
```
    initCard (scrollTop) {
      scrollTop = scrollTop || 0
      this.$nextTick(() => {
        // 视野小卡片的宽，单位为个，card宽度为280
        let visibleCardW = Math.floor((this.$refs.cardBox.getBoundingClientRect().width - 55) / this.cardW)
        // 保存小卡片数量的宽
        this.visibleCardW = visibleCardW
        // 视野小卡片数量的高，单位为个，以最小卡片高度200px为最小高度
        let visibleCardH = Math.ceil((this.cardHeight) / this.cardH)
        // 视野小卡片的数量， 单位为个，宽 * 高
        let visibleCardCount = visibleCardW * visibleCardH
        // 加载的第一个小卡片
        let start = Math.floor((scrollTop / this.cardH)) * visibleCardW
        // 保存第一个小卡片的数
        this.cardStart = start
        // 加载的最后一个小卡片, 加visibleCardW的原因是多加载一行，减少视觉不适
        let end = start + visibleCardCount + visibleCardW
        console.log('加载的第一个小卡片', start)
        console.log('加载的最后一个小卡片', end)
        this.makeDeviceLengthArr()
        this.updateVisibleCardData(start, end)
      })
    },
```
`2`. 构造所有设备数据长度的数组，用于视图小卡片数据的处理
```
    makeDeviceLengthArr () {
      // 各种设备长度的数组
      this.deviceLengthArr = []
      // 分两种情况，搜索状态和无搜索状态
      if (this.deviceCodeSearch) {
        this.deviceLengthArr.push(this.searchDeviceData.length)
      } else {
        // 把各种设备的长度存进数组
        this.deviceLengthArr.push(this.alarmDeviceData.length)
        this.deviceLengthArr.push(this.warningDeviceData.length)
        this.deviceLengthArr.push(this.electricBoxdata.length)
        this.deviceLengthArr.push(this.detectorData.length)
        this.deviceLengthArr.push(this.breakerCardData.length)
        this.deviceLengthArr.push(this.voltmeterData.length)
        this.deviceLengthArr.push(this.smokeDetectorData.length)
      }
      console.log('我现在的设备长度数组是', this.deviceLengthArr)
    },
```
`3`. 更新视图小卡片的数据, 处理过渡状态
```
    updateVisibleCardData (start, end) {
      // 设备长度的累加
      var deviceLength = 0
      // 开始渲染的设备类型中的第几个
      var deviceFirst = 0
      //  保存参数start, 这是计算来的卡片开始数，未减去换行
      var theStart = start
      for (let i = 0, n = this.deviceLengthArr.length; i < n; i++) {
        if (theStart < deviceLength + this.deviceLengthArr[i]) {
          deviceFirst = theStart - deviceLength
          for (let j = i; j < n; j++) {
            if (end <= deviceLength + this.deviceLengthArr[j]) {
              // 控制页面刷新，开放
              this.stopUpdate = false
              this.structureData(i, deviceFirst, j, end - deviceLength)
              break
            }
            if (j === n - 1 && end >= this.deviceLengthArr[j] - 1) {
              // 验证是否锁定
              if (this.stopUpdate === true) {
                return
              }
              this.structureData(i, deviceFirst, j, this.deviceLengthArr[j] - 1)
              // 控制刷新页面，锁定
              this.stopUpdate = true
              break
            }
            deviceLength += this.deviceLengthArr[j]
          }
          break
        }
        deviceLength += this.deviceLengthArr[i]
        // 减去换行的空白卡片数, 重要！！！！，不然会丢失卡片
        theStart = theStart - ((this.deviceLengthArr[i] % this.visibleCardW) > 0 ? (this.visibleCardW - (this.deviceLengthArr[i] % this.visibleCardW)) : 0)
        end = end - ((this.deviceLengthArr[i] % this.visibleCardW) > 0 ? (this.visibleCardW - (this.deviceLengthArr[i] % this.visibleCardW)) : 0)
      }
    },
```
`4`. 构造视图页面，就是把用户应该看到的数据渲染到页面上
```
    structureData (firstType, typeIndex, endType, typeEnd) {
      // 在某种类设备类型的第几个开始渲染
      var theTypeIndex = typeIndex
      // 保存开始的设备类型，以免受到下面计算的影响
      var theFirstType = firstType
      // 初始化视图数据数组
      this.visibleCardDataArr = [[], [], [], [], [], [], []]
      for (; firstType <= endType; firstType++) {
        if (firstType === endType) {
          // 如果是搜索状态的话，直接把用户视野数据放在第一个下标
          if (this.deviceCodeSearch) {
            Vue.set(this.visibleCardDataArr, firstType, this.searchDeviceData.slice(theTypeIndex, typeEnd + 1))
            break
          }
          Vue.set(this.visibleCardDataArr, firstType, this.changeIntoDevice(firstType).slice(theTypeIndex, typeEnd + 1))
          break
        } else if (firstType === endType - 1) {
          Vue.set(this.visibleCardDataArr, firstType, this.changeIntoDevice(firstType).slice(theTypeIndex))
          Vue.set(this.visibleCardDataArr, endType, this.changeIntoDevice(endType).slice(0, typeEnd + 1))
          break
        } else {
          Vue.set(this.visibleCardDataArr, firstType, this.changeIntoDevice(firstType).slice(theTypeIndex))
          theTypeIndex = 0
        }
      }
      this.$nextTick(() => {
        this.cardBoxTitle = [true, true, true, true, true, true, true]
        if (typeIndex !== 0) {
          this.cardBoxTitle[theFirstType] = false
        }
        this.$refs.content.style.webkitTransform = `translate3d(0, ${(this.cardStart / this.visibleCardW) * this.cardH}px, 0)` // 把可见区域的 top 设置为起始元素在整个列表中的位置（使用 transform 是为了更好的性能）
      })
    },
```

##　三　进一步优化

１．做数据缓存处理
２．小卡片的高度动态计算

。。。大概是留给你们了，我们已经老了
  [1]: https://jsfiddle.net/furybean/mnkfq0xm/45/