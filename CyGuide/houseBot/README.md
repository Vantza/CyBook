# housebot
 a bot for house info from lianjia

# Environment
* Scrapy       : 1.7.3
* lxml         : 4.4.1.0
* libxml2      : 2.9.9
* cssselect    : 1.1.0
* parsel       : 1.5.2
* w3lib        : 1.21.0
* Twisted      : 19.7.0
* Python       : 3.7.4 (default, Sep  7 2019, 18:27:02) - [Clang 10.0.1 (clang-1001.0.46.4)]
* pyOpenSSL    : 19.0.0 (OpenSSL 1.1.1c  28 May 2019)
* cryptography : 2.7
* Platform     : Darwin-18.7.0-x86_64-i386-64bit

# How
* start mongoDB service, here use docker compose to start a mongoDB
    - docker-compose -f mongo.yml up
    
* start scrape data from lianjia
    - scrapy crawl lianjia-spider

* find data in houseDB.houseInfo