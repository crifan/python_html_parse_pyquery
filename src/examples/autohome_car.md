# 汽车之家品牌车型车系

下面贴出，用PySpider爬取汽车之家的车型车系数据的代码，其中包含`PyQuery`相关代码，供参考。

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2018-04-27 21:53:02
# Project: autohomeBrandData

from pyspider.libs.base_handler import *
import string
import re

class Handler(BaseHandler):
    crawl_config = {
    }

    # @every(minutes=24 * 60)
    def on_start(self):
        for eachLetter in list(string.ascii_lowercase):
            self.crawl("https://www.autohome.com.cn/grade/carhtml/%s.html" % eachLetter, callback=self.gradCarHtmlPage)

    @catch_status_code_error
    def gradCarHtmlPage(self, response):
        print("gradCarHtmlPage: response=", response)
        picSeriesItemList = response.doc('.rank-list-ul li div a[href*="/pic/series"]').items()
        print("picSeriesItemList=", picSeriesItemList)
        # print("len(picSeriesItemList)=%s"%(len(picSeriesItemList)))
        for each in picSeriesItemList:
            self.crawl(each.attr.href, callback=self.picSeriesPage)

    @config(priority=2)
    def picSeriesPage(self, response):
        # &lt;a href="/pic/series-t/66.html"&gt;查看停产车型&amp;nbsp;&amp;gt;&lt;/a&gt;
        # &lt;a class="ckmore" href="/pic/series/588.html"&gt;查看在售车型&amp;nbsp;&amp;gt;&lt;/a&gt;
        # &lt;span class="fn-right"&gt;&amp;nbsp;&lt;/span&gt;
        fnRightPicSeries = response.doc('.search-pic-tbar .fn-right a[href*="/pic/series"]')
        print("fnRightPicSeries=", fnRightPicSeries)
        if fnRightPicSeries:
            # hrefValue = fnRightPicSeries.attr.href
            # print("hrefValue=", hrefValue)
            # fullPicSeriesUrl = "https://car.autohome.com.cn" + hrefValue
            fullPicSeriesUrl = fnRightPicSeries.attr.href
            print("fullPicSeriesUrl=", fullPicSeriesUrl)
            self.crawl(fullPicSeriesUrl, callback=self.picSeriesPage)

        # contine parse brand data
        aDictList = []
        # for eachA in response.doc('.breadnav a[href^="/"]').items():
        for eachA in response.doc('.breadnav a[href*="/pic/"]').items():
            eachADict = {
                "text" : eachA.text(),
                "href": eachA.attr.href
            }
            print("eachADict=", eachADict)
            aDictList.append(eachADict)

        print("aDictList=", aDictList)

        mainBrandDict = aDictList[-3]
        subBrandDict = aDictList[-2]
        brandSerieDict = aDictList[-1]
        print("mainBrandDict=%s, subBrandDict=%s, brandSerieDict=%s"%(mainBrandDict, subBrandDict, brandSerieDict))
        
        dtTextList = []
        for eachDt in response.doc("dl.search-pic-cardl dt").items():
            dtTextList.append(eachDt.text())
        
        print("dtTextList=", dtTextList)

        groupCount = len(dtTextList)
        print("groupCount=", groupCount)

        for eachDt in response.doc("dl.search-pic-cardl dt").items():
            dtTextList.append(eachDt.text())

        ddUlEltList = []
        for eachDdUlElt in response.doc("dl.search-pic-cardl dd ul").items():
            ddUlEltList.append(eachDdUlElt)
        
        print("ddUlEltList=", ddUlEltList)

        modelDetailDictList = []
        
        for curIdx in range(groupCount):
            curGroupTitle = dtTextList[curIdx]
            print("------[%d] %s" % (curIdx, curGroupTitle))
            
            for eachLiAElt in ddUlEltList[curIdx].items("li a"):
                # 1. model name
                # curModelName = eachLiAElt.text()
                curModelName = eachLiAElt.contents()[0]
                curModelName = curModelName.strip()
                print("curModelName=", curModelName)
                curFullModelName = curGroupTitle + " " + curModelName
                print("curFullModelName=", curFullModelName)
                
                # 2. model id + carSeriesId + spec url
                curModelId = ""
                curSeriesId = ""
                curModelSpecUrl = ""
                modelSpecUrlTemplate = "https://www.autohome.com.cn/spec/%s/#pvareaid=2042128"
                curModelPicUrl = eachLiAElt.attr.href
                print("curModelPicUrl=", curModelPicUrl)
                #https://car.autohome.com.cn/pic/series-s32708/3457.html#pvareaid=2042220
                foundModelSeriesId = re.search("pic/series-s(?P&lt;curModelId&gt;\d+)/(?P&lt;curSeriesId&gt;\d+)\.html", curModelPicUrl)
                print("foundModelSeriesId=", foundModelSeriesId)
                if foundModelSeriesId:
                    curModelId = foundModelSeriesId.group("curModelId")
                    curSeriesId = foundModelSeriesId.group("curSeriesId")
                    print("curModelId=%s, curSeriesId=%s", curModelId, curSeriesId)
                    curModelSpecUrl = (modelSpecUrlTemplate) % (curModelId)
                    print("curModelSpecUrl=", curModelSpecUrl)
                
                # 3. model status
                modelStatus = "在售"
                foundStopSale = eachLiAElt.find('i[class*="icon-stopsale"]')
                if foundStopSale:
                    modelStatus = "停售"
                else:
                    foundWseason = eachLiAElt.find('i[class*="icon-wseason"]')
                    if foundWseason:
                        modelStatus = "未上市"

                modelDetailDictList.append({
                    "url": curModelSpecUrl,
                    "车系ID": curSeriesId,
                    "车型ID": curModelId,
                    "车型": curFullModelName,
                    "状态": modelStatus
                })
        print("modelDetailDictList=", modelDetailDictList)

        allSerieDictList = []
        for curIdx, eachModelDetailDict in enumerate(modelDetailDictList):
            curSerieDict = {
                "品牌": mainBrandDict["text"],
                "子品牌": subBrandDict["text"],
                "车系": brandSerieDict["text"],
                "车系ID": eachModelDetailDict["车系ID"],
                "车型": eachModelDetailDict["车型"],
                "车型ID": eachModelDetailDict["车型ID"],
                "状态": eachModelDetailDict["状态"]
            }
            allSerieDictList.append(curSerieDict)
            # print("before send_message: [%d] curSerieDict=%s" % (curIdx, curSerieDict))
            # self.send_message(self.project_name, curSerieDict, url=eachModelDetailDict["url"])
            print("[%d] curSerieDict=%s" % (curIdx, curSerieDict))
            self.crawl(eachModelDetailDict["url"], callback=self.carModelSpecPage, save=curSerieDict)

        # print("allSerieDictList=", allSerieDictList)
        # return allSerieDictList

    #def on_message(self, project, msg):
    #    print("on_message: msg=", msg)
    #    return msg
    
    @catch_status_code_error
    def carModelSpecPage(self, response):
        print("carModelSpecPage: response=", response)
        # https://www.autohome.com.cn/spec/32708/#pvareaid=2042128
        curSerieDict = response.save
        print("curSerieDict", curSerieDict)

        # cityDealerPriceInt = 0
        # cityDealerPriceElt = response.doc('.cardetail-infor-price #cityDealerPrice span span[class*="price"]')
        # print("cityDealerPriceElt=%s" % cityDealerPriceElt)
        # if cityDealerPriceElt:
        #     cityDealerPriceFloatStr = cityDealerPriceElt.text()
        #     print("cityDealerPriceFloatStr=", cityDealerPriceFloatStr)
        #     cityDealerPriceFloat = float(cityDealerPriceFloatStr)
        #     print("cityDealerPriceFloat=", cityDealerPriceFloat)
        #     cityDealerPriceInt = int(cityDealerPriceFloat * 10000)
        #     print("cityDealerPriceInt=", cityDealerPriceInt)

        msrpPriceInt = 0
        # body &gt; div.content &gt; div.row &gt; div.column.grid-16 &gt; div.cardetail.fn-clear &gt; div.cardetail-infor &gt; div.cardetail-infor-price.fn-clear &gt; ul &gt; li.li-price.fn-clear &gt; span
        # 厂商指导价=厂商建议零售价格=MSRP=Manufacturer's suggested retail price
        msrpPriceElt = response.doc('.cardetail-infor-price li[class*="li-price"] span[data-price]')
        print("msrpPriceElt=", msrpPriceElt)
        if msrpPriceElt:
            msrpPriceStr = msrpPriceElt.attr("data-price")
            print("msrpPriceStr=", msrpPriceStr)
            foundMsrpPrice = re.search("(?P&lt;msrpPrice&gt;[\d\.]+)万元", msrpPriceStr)
            print("foundMsrpPrice=", foundMsrpPrice)
            if foundMsrpPrice:
                msrpPrice = foundMsrpPrice.group("msrpPrice")
                print("msrpPrice=", msrpPrice)
                msrpPriceFloat = float(msrpPrice)
                print("msrpPriceFloat=", msrpPriceFloat)
                msrpPriceInt = int(msrpPriceFloat * 10000)
                print("msrpPriceInt=", msrpPriceInt)

        # curSerieDict["经销商参考价"] = cityDealerPriceInt
        curSerieDict["厂商指导价"] = msrpPriceInt        

        return curSerieDict
```

详见：

[【已解决】写Python爬虫爬取汽车之家品牌车系车型数据](http://www.crifan.com/use_pyspider_to_crawl_autohome_car_brand_serial_model_data)
