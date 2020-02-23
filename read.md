是不是总在斗图中落下风，是不是总是觉得没有你想要的表情包，是不是总有些文字不知道如何发出去而不尴尬。没关系，我来了，那个疯一样的男人，它来了，带着标签包来了。5分钟让你成为表情包之王。哦，对了，声明一下，爬虫知识仅用于教学，不可做损害他人利益和违法犯罪的事情哦！

1、安装环境

首先安装Node，没有安装，请出门左转。

2、创建js文件

新建1个目录，在里面创建1个index.js文件，创建1个img目录。如图所示

3、安装依赖包

npm install axios --save

npm install cheerio --save

axios用来请求数据，cheerio用来解析出表情包的链接。对这两个库不熟悉的同学，可以看一下这2个库的文档，或者看我的教学视频。

4、index.js文件导入包

const cheerio = require("cheerio");

const axios = require('axios')

const fs = require('fs')

const path = require('path')

cheerio用来解析html获取，表情图片链接。

axios用来发送请求，获取HTML页面，和下载图片。

fs模块用来将表情图片写入到磁盘。

path用来解析路径。

5、分析爬取表情包的网站

这次爬取表情包的网站案例是：

https://www.doutula.com/

网页点击右键，查看网页源代码，可以看到表情列表页，分页的链接在这里

继续分析，列表页中，每个具体的表情包页面链接是：

点开每个链接，在查看源代码，发现每个页面中的表情包图片链接是：

6、爬取表情包逻辑

通过分页，获取每个表情包的列表页，在通过列表页获取所有表情包页面，在将所有表情包页面的表情图片链接地址获取，最后通过表情包链接地址，下载所有的图片。

7、代码实现

获取具体的列表页面总数

//获取页面总数

async function getNum(){

    res = await axios.get(httpUrl)

    let $ = cheerio.load(res.data)

    let btnLength = $('.pagination li').length;

    let allNum = $('.pagination li').eq(btnLength-2).find('a').text()

    //console.log(allNum)

    return allNum

} 

获取列表页，并解析出列表页链接

async function getListPage(pageNum){

    let httpUrl = "https://www.doutula.com/article/list/?page="+pageNum;

    let res = await axios.get(httpUrl)

    //console.log(res.data)

    //cheerio解析html文档

    let $ = cheerio.load(res.data)

    //获取当前页面的所有的表情页面的链接

    $('#home .col-sm-9>a').each(async (i,element)=>{

       let  pageUrl = $(element).attr('href');

       let title = $(element).find('.random_title').text()

       let reg = /(.*?)\d/igs;

       title = reg.exec(title)[1];

       fs.mkdir('./img/'+title,function(err){

            if(err){

                //console.log(err)

            }else{

                console.log("成功创建目录："+'./img/'+title)

            }

       });

       //console.log(title)

       await wait(100);

       parsePage(pageUrl,title)

    })

}

解析表情包页面，并下载图片

async function parsePage(pageUrl,title){

    let res = await axios.get(pageUrl);

    let $ = cheerio.load(res.data)

    $('.pic-content img').each(async (i,element)=>{

        let imgUrl = $(element).attr('src')

       

        //console.log(path.parse(imgUrl))

        extName = path.extname(imgUrl)

        //图片写入的路径和名字

        await wait(50);

        let imgPath = `./img/${title}/${title}-${i}${extName}`

        //创建写入图片流

        let ws = fs.createWriteStream(imgPath)

        axios.get(imgUrl,{responseType:'stream'}).then(function(res){

            res.data.pipe(ws)

            console.log("图片加载完成："+imgPath)

            //关闭写入流

            res.data.on('close',function(){

                ws.close()

            })

        })

        

    })

}

调用爬取函数，和控制爬取节奏，避免爬取速度太快而对服务器造成影响和察觉。

let httpUrl = "https://www.doutula.com/article/list/?page=1"

//等待函数

async function wait(milliseconds){

    return new Promise(function(resolve,reject){

        setTimeout(function(){

            resolve()

        },milliseconds)

    })

}

async function spider(){

    //获取所有的页面总数

    let allPageNum = await getNum()

    for(let i=1;i<=allPageNum;i++){

        await wait(2000)

        getListPage(i)

    }

}

spider()

8、最终效果

