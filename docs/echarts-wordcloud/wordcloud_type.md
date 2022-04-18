# echarts自定义词云图的一种方式

直接上代码：




```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <script src='https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js'></script>
        <!-- <script src="../../echarts/dist/echarts.js"></script> -->
        <script src='../dist/echarts-wordcloud.js'></script>
    </head>
<body>
  <div id="main" style="width: 700px; height: 500px"></div>
  <script>
    onload = function () {
      var data = {
          //这是数据示例，自行添加，格式参考如下即可
        value: [
          {
            "name": "手相",
            "value": 32
          },
          {
            "name": "公益",
            "value": 90
          }
        ],
          ////掩码图像。设置形状为图片的形状。想要什么样的云图效果，自己制作一个/掩码图像的png图片即可。这也是一个示例
          image:"data:image/png;base64,....."
          }
      var myChart = echarts.init(document.getElementById('main'));
      var maskImage = new Image();
      maskImage.src = data.image

      maskImage.onload = function(){
        myChart.setOption( {
          backgroundColor:'#fff',
          tooltip: {
            show: false
          },
          series: [{
             type: 'wordCloud',
			 //     shape: 'circle',
			//圆形circle（默认），心形（heart，apple），菱形（diamond），
			//向前的三角形（triangle-forward） ，三角形（triangle），直立三角形（triangle-upright), 五边形（pentagon）,
           //maskImage的横纵比为1:1
       		keepAspect: false,
           // 词间距，数值越小，间距越小，这里间距太小的话，会出现大词把小词套住的情况，比如一个大的口字，中间会有比较大的空隙，这时候他会把一些很小的字放在口字里面，这样的话，鼠标就无法选中里面的那个小字
			gridSize: 8,
		// 允许词太大的时候，超出画布的范围
			//是否允许词云在边界外渲染，直接使用默认参数 false 就可以，否则容易造成词重叠。
			drawOutOfBound: false,
			// 布局的时候是否有动画
			layoutAnimation: true,
           //sizeRange ：词云的文字字号范围，默认是[12, 60] ，
            //词云会根据提供原始数据的 value 对文字的字号进行渲染。以默认值为例， value 最小的渲染为 12px ，最大的渲染为 60px ，中间的值按比例计算相应的数值。
            sizeRange: [5, 50],
            //词云中文字的角度，词云中的文字会随机的在 rotationRange 范围内旋转角度，
            //渲染的梯度就是 rotationStep ，这个值越小，词云里出现的角度种类就越多
            rotationRange: [-90, 90],
            rotationStep: 45, 
              //词云中文字的样式， textStyle是初始的样式， emphasis 是鼠标移到文字上的样式
 
            maskImage: maskImage,
                    textStyle: {
                        color: function () {
                            return 'rgb(' + [
                                Math.round(Math.random() * 160),
                                Math.round(Math.random() * 160),
                                Math.round(Math.random() * 160)
                            ].join(',') + ')';
                        }
                    },
                    emphasis: {
						            focus: 'self',

						            textStyle: {
						                textShadowBlur: 10,
						                textShadowColor: '#333'
						            }
						        },
            left: 'center',
            top: 'center',
            right: null,
            bottom: null,
            data: data.value
          }]
        })
      }

    }
  </script>
</body>
</html>


```

图像base64参考：

```
 image:"data:image/png;base64,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"
          
```

数据参考：

```
{
	          "name": "花鸟市场",
	          "value": 1446
	        },
          {
            "name": "汽车",
            "value": 928
          },
          {
            "name": "视频",
            "value": 906
          },
          {
            "name": "电视",
            "value": 825
          },
          {
            "name": "Lover Boy 88",
            "value": 514
          },
          {
            "name": "动漫",
            "value": 486
          },
          {
            "name": "音乐",
            "value": 53
          },
          {
            "name": "直播",
            "value": 163
          },
          {
            "name": "广播电台",
            "value": 86
          },
          {
            "name": "戏曲曲艺",
            "value": 17
          },
          {
            "name": "演出票务",
            "value": 6
          },
          {
            "name": "给陌生的你听",
            "value": 1
          },
          {
            "name": "资讯",
            "value": 1437
          },
          {
            "name": "商业财经",
            "value": 422
          },
          {
            "name": "娱乐八卦",
            "value": 353
          },
          {
            "name": "军事",
            "value": 331
          },
          {
            "name": "科技资讯",
            "value": 313
          },
          {
            "name": "社会时政",
            "value": 307
          },
          {
            "name": "时尚",
            "value": 43
          },
          {
            "name": "网络奇闻",
            "value": 15
          },
          {
            "name": "旅游出行",
            "value": 438
          },
          {
            "name": "景点类型",
            "value": 957
          },
          {
            "name": "国内游",
            "value": 927
          },
          {
            "name": "远途出行方式",
            "value": 908
          },
          {
            "name": "酒店",
            "value": 693
          },
          {
            "name": "关注景点",
            "value": 611
          },
          {
            "name": "旅游网站偏好",
            "value": 512
          },
          {
            "name": "出国游",
            "value": 382
          },
          {
            "name": "交通票务",
            "value": 312
          },
          {
            "name": "旅游方式",
            "value": 187
          },
          {
            "name": "旅游主题",
            "value": 163
          },
          {
            "name": "港澳台",
            "value": 104
          },
          {
            "name": "本地周边游",
            "value": 3
          },
          {
            "name": "小卖家",
            "value": 1331
          },
          {
            "name": "全日制学校",
            "value": 941
          },
          {
            "name": "基础教育科目",
            "value": 585
          },
          {
            "name": "考试培训",
            "value": 473
          },
          {
            "name": "语言学习",
            "value": 358
          },
          {
            "name": "留学",
            "value": 246
          },
          {
            "name": "K12课程培训",
            "value": 207
          },
          {
            "name": "艺术培训",
            "value": 194
          },
          {
            "name": "技能培训",
            "value": 104
          },
          {
            "name": "IT培训",
            "value": 87
          },
          {
            "name": "高等教育专业",
            "value": 63
          },
          {
            "name": "家教",
            "value": 48
          },
          {
            "name": "体育培训",
            "value": 23
          },
          {
            "name": "职场培训",
            "value": 5
          },
          {
            "name": "金融财经",
            "value": 1328
          },
          {
            "name": "银行",
            "value": 765
          },
          {
            "name": "股票",
            "value": 452
          },
          {
            "name": "保险",
            "value": 415
          },
          {
            "name": "贷款",
            "value": 253
          },
          {
            "name": "基金",
            "value": 211
          },
          {
            "name": "信用卡",
            "value": 180
          },
          {
            "name": "外汇",
            "value": 138
          },
          {
            "name": "P2P",
            "value": 116
          },
          {
            "name": "贵金属",
            "value": 98
          },
          {
            "name": "债券",
            "value": 93
          },
          {
            "name": "网络理财",
            "value": 92
          },
          {
            "name": "信托",
            "value": 90
          },
          {
            "name": "征信",
            "value": 76
          },
          {
            "name": "期货",
            "value": 76
          },
          {
            "name": "公积金",
            "value": 40
          },
          {
            "name": "银行理财",
            "value": 36
          },
          {
            "name": "银行业务",
            "value": 30
          },
          {
            "name": "典当",
            "value": 7
          },
          {
            "name": "海外置业",
            "value": 1
          },
          {
            "name": "汽车",
            "value": 1309
          },
          {
            "name": "汽车档次",
            "value": 965
          },
          {
            "name": "汽车品牌",
            "value": 900
          },
          {
            "name": "汽车车型",
            "value": 727
          },
          {
            "name": "购车阶段",
            "value": 461
          },
          {
            "name": "二手车",
            "value": 309
          },
          {
            "name": "汽车美容",
            "value": 260
          },
          {
            "name": "新能源汽车",
            "value": 173
          },
          {
            "name": "汽车维修",
            "value": 155
          },
          {
            "name": "租车服务",
            "value": 136
          },
          {
            "name": "车展",
            "value": 121
          },
          {
            "name": "违章查询",
            "value": 76
          },
          {
            "name": "汽车改装",
            "value": 62
          },
          {
            "name": "汽车用品",
            "value": 37
          },
          {
            "name": "路况查询",
            "value": 32
          },
          {
            "name": "汽车保险",
            "value": 28
          },
          {
            "name": "陪驾代驾",
            "value": 4
          },
          {
            "name": "网络购物",
            "value": 1275
          },
          {
            "name": "做我的猫",
            "value": 1088
          },
          {
            "name": "只想要你知道",
            "value": 907
          },
          {
            "name": "团购",
            "value": 837
          },
          {
            "name": "比价",
            "value": 201
          },
          {
            "name": "海淘",
            "value": 195
          },
          {
            "name": "移动APP购物",
            "value": 179
          },
          {
            "name": "支付方式",
            "value": 119
          },
          {
            "name": "代购",
            "value": 43
          },
          {
            "name": "体育健身",
            "value": 1234
          },
          {
            "name": "体育赛事项目",
            "value": 802
          },
          {
            "name": "运动项目",
            "value": 405
          },
          {
            "name": "体育类赛事",
            "value": 337
          },
          {
            "name": "健身项目",
            "value": 199
          },
          {
            "name": "健身房健身",
            "value": 78
          },
          {
            "name": "运动健身",
            "value": 77
          },
          {
            "name": "家庭健身",
            "value": 36
          },
          {
            "name": "健身器械",
            "value": 29
          },
          {
            "name": "办公室健身",
            "value": 3
          },
          {
            "name": "商务服务",
            "value": 1201
          },
          {
            "name": "法律咨询",
            "value": 508
          },
          {
            "name": "化工材料",
            "value": 147
          },
          {
            "name": "广告服务",
            "value": 125
          },
          {
            "name": "会计审计",
            "value": 115
          },
          {
            "name": "人员招聘",
            "value": 101
          },
          {
            "name": "印刷打印",
            "value": 66
          },
          {
            "name": "知识产权",
            "value": 32
          },
          {
            "name": "翻译",
            "value": 22
          },
          {
            "name": "安全安保",
            "value": 9
          },
          {
            "name": "公关服务",
            "value": 8
          },
          {
            "name": "商旅服务",
            "value": 2
          },
          {
            "name": "展会服务",
            "value": 2
          },
          {
            "name": "特许经营",
            "value": 1
          },
          {
            "name": "休闲爱好",
            "value": 1169
          },
          {
            "name": "收藏",
            "value": 412
          },
          {
            "name": "摄影",
            "value": 393
          },
          {
            "name": "温泉",
            "value": 230
          },
          {
            "name": "博彩彩票",
            "value": 211
          },
          {
            "name": "美术",
            "value": 207
          },
          {
            "name": "书法",
            "value": 139
          },
          {
            "name": "DIY手工",
            "value": 75
          },
          {
            "name": "舞蹈",
            "value": 23
          },
          {
            "name": "钓鱼",
            "value": 21
          },
          {
            "name": "棋牌桌游",
            "value": 17
          },
          {
            "name": "KTV",
            "value": 6
          },
          {
            "name": "密室",
            "value": 5
          },
          {
            "name": "采摘",
            "value": 4
          },
          {
            "name": "电玩",
            "value": 1
          },
          {
            "name": "真人CS",
            "value": 1
          },
          {
            "name": "轰趴",
            "value": 1
          },
          {
            "name": "家电数码",
            "value": 1111
          },
          {
            "name": "手机",
            "value": 885
          },
          {
            "name": "电脑",
            "value": 543
          },
          {
            "name": "大家电",
            "value": 321
          },
          {
            "name": "家电关注品牌",
            "value": 253
          },
          {
            "name": "网络设备",
            "value": 162
          },
          {
            "name": "摄影器材",
            "value": 149
          },
          {
            "name": "影音设备",
            "value": 133
          },
          {
            "name": "办公数码设备",
            "value": 113
          },
          {
            "name": "生活电器",
            "value": 67
          },
          {
            "name": "厨房电器",
            "value": 54
          },
          {
            "name": "智能设备",
            "value": 45
          },
          {
            "name": "个人护理电器",
            "value": 22
          },
          {
            "name": "服饰鞋包",
            "value": 1047
          },
          {
            "name": "服装",
            "value": 566
          },
          {
            "name": "饰品",
            "value": 289
          },
          {
            "name": "鞋",
            "value": 184
          },
          {
            "name": "箱包",
            "value": 168
          },
          {
            "name": "奢侈品",
            "value": 137
          },
          {
            "name": "母婴亲子",
            "value": 1041
          },
          {
            "name": "孕婴保健",
            "value": 505
          },
          {
            "name": "母婴社区",
            "value": 299
          },
          {
            "name": "早教",
            "value": 103
          },
          {
            "name": "奶粉辅食",
            "value": 66
          },
          {
            "name": "童车童床",
            "value": 41
          },
          {
            "name": "关注品牌",
            "value": 271
          },
          {
            "name": "宝宝玩乐",
            "value": 30
          },
          {
            "name": "母婴护理服务",
            "value": 25
          },
          {
            "name": "纸尿裤湿巾",
            "value": 16
          },
          {
            "name": "妈妈用品",
            "value": 15
          },
          {
            "name": "宝宝起名",
            "value": 12
          },
          {
            "name": "童装童鞋",
            "value": 9
          },
          {
            "name": "胎教",
            "value": 8
          },
          {
            "name": "宝宝安全",
            "value": 1
          },
          {
            "name": "宝宝洗护用品",
            "value": 1
          },
          {
            "name": "软件应用",
            "value": 1018
          },
          {
            "name": "系统工具",
            "value": 896
          },
          {
            "name": "理财购物",
            "value": 440
          },
          {
            "name": "生活实用",
            "value": 365
          },
          {
            "name": "影音图像",
            "value": 256
          },
          {
            "name": "社交通讯",
            "value": 214
          },
          {
            "name": "手机美化",
            "value": 39
          },
          {
            "name": "办公学习",
            "value": 28
          },
          {
            "name": "应用市场",
            "value": 23
          },
          {
            "name": "母婴育儿",
            "value": 14
          },
          {
            "name": "游戏",
            "value": 946
          },
          {
            "name": "手机游戏",
            "value": 565
          },
          {
            "name": "PC游戏",
            "value": 353
          },
          {
            "name": "网页游戏",
            "value": 254
          },
          {
            "name": "游戏机",
            "value": 188
          },
          {
            "name": "模拟辅助",
            "value": 166
          },
          {
            "name": "个护美容",
            "value": 942
          },
          {
            "name": "护肤品",
            "value": 177
          },
          {
            "name": "彩妆",
            "value": 133
          },
          {
            "name": "美发",
            "value": 80
          },
          {
            "name": "香水",
            "value": 50
          },
          {
            "name": "个人护理",
            "value": 46
          },
          {
            "name": "美甲",
            "value": 26
          },
          {
            "name": "SPA美体",
            "value": 21
          },
          {
            "name": "花鸟萌宠",
            "value": 914
          },
          {
            "name": "绿植花卉",
            "value": 311
          },
          {
            "name": "狗",
            "value": 257
          },
          {
            "name": "其他宠物",
            "value": 131
          },
          {
            "name": "水族",
            "value": 125
          },
          {
            "name": "猫",
            "value": 122
          },
          {
            "name": "动物",
            "value": 81
          },
          {
            "name": "鸟",
            "value": 67
          },
          {
            "name": "宠物用品",
            "value": 41
          },
          {
            "name": "宠物服务",
            "value": 26
          },
          {
            "name": "书籍阅读",
            "value": 913
          },
          {
            "name": "网络小说",
            "value": 483
          },
          {
            "name": "关注书籍",
            "value": 128
          },
          {
            "name": "文学",
            "value": 105
          },
          {
            "name": "报刊杂志",
            "value": 77
          },
          {
            "name": "人文社科",
            "value": 22
          },
          {
            "name": "建材家居",
            "value": 907
          },
          {
            "name": "装修建材",
            "value": 644
          },
          {
            "name": "家具",
            "value": 273
          },
          {
            "name": "家居风格",
            "value": 187
          },
          {
            "name": "家居家装关注品牌",
            "value": 140
          },
          {
            "name": "家纺",
            "value": 107
          },
          {
            "name": "厨具",
            "value": 47
          },
          {
            "name": "灯具",
            "value": 43
          },
          {
            "name": "家居饰品",
            "value": 29
          },
          {
            "name": "家居日常用品",
            "value": 10
          },
          {
            "name": "生活服务",
            "value": 883
          },
          {
            "name": "物流配送",
            "value": 536
          },
          {
            "name": "家政服务",
            "value": 108
          },
          {
            "name": "摄影服务",
            "value": 49
          },
          {
            "name": "搬家服务",
            "value": 38
          },
          {
            "name": "物业维修",
            "value": 37
          },
          {
            "name": "婚庆服务",
            "value": 24
          },
          {
            "name": "二手回收",
            "value": 24
          },
          {
            "name": "鲜花配送",
            "value": 3
          },
          {
            "name": "维修服务",
            "value": 3
          },
          {
            "name": "殡葬服务",
            "value": 1
          },
          {
            "name": "求职创业",
            "value": 874
          },
          {
            "name": "创业",
            "value": 363
          },
          {
            "name": "目标职位",
            "value": 162
          },
          {
            "name": "目标行业",
            "value": 50
          },
          {
            "name": "兼职",
            "value": 21
          },
          {
            "name": "期望年薪",
            "value": 20
          },
          {
            "name": "实习",
            "value": 16
          },
          {
            "name": "雇主类型",
            "value": 10
          },
          {
            "name": "星座运势",
            "value": 789
          },
          {
            "name": "星座",
            "value": 316
          },
          {
            "name": "算命",
            "value": 303
          },
          {
            "name": "解梦",
            "value": 196
          },
          {
            "name": "风水",
            "value": 93
          },
          {
            "name": "面相分析",
            "value": 47
          },
          {
            "name": "手相",
            "value": 32
          },
```

