# Wikimedia search

A sample project to show some Clojure skills and have a grasp about the Liberator performance dealing with a high volume of concurrent requets. 

# Challenge
Wikimedia publish article abstracts as XML at [http://dumps.wikimedia.org/enwiki](http://dumps.wikimedia.org/enwiki) The files are very large but there is a smaller one (83Mb) at [latest](http://dumps.wikimedia.org/enwiki/latest/enwiki-latest-abstract23.xml) you can use for the test.

Each file contains a number of abstracts structured as per the following example



```
<feed>
	<doc>
		<title>Wikipedia: IFK Holmsund</title>
		<url>http://en.wikipedia.org/wiki/IFK_Holmsund</url>
		<abstract>Idrottsföreningen Kamraterna Holmsund was a Swedish football team from Holmsund, Västerbotten County founded in 1923 http://www.bolletinen.</abstract>
		￼￼￼￼<links>
			<sublink linktype="nav"><anchor>Notable players</anchor>
				<link>http://en.wikipedia.org/wiki/IFK_Holmsund#Notable_players</link></sublink>
			<sublink linktype="nav"><anchor>References</anchor>
				<link>http://en.wikipedia.org/wiki/IFK_Holmsund#References</link></sublink>
		</links>
	</doc>
	<!—- and so on -->
</feed>

````



We would like you to please write a system that reads one of these files and stores it in a form that allows you to search the file using keywords from the abstract and ttle.


Once you have that, you should create a simple JSON API that returns the title, url and abstract for all the documents matching a query.


For example, if you submit query that like this `GET http://my.techtest.example.com/search?q=Holmsund`
 you should get back a JSON document that looks like


```
{
  "q":"Holmsund",
  "results":[
    {
      "title":"Wikipedia: IFK Holmsund",
      "url":"http:\/\/en.wikipedia.org\/wiki\/IFK_Holmsund",
      "abstract":"Idrottsf\u00f6reningen Kamraterna Holmsund
was a Swedish football team from Holmsund, V\u00e4sterbotten
County founded in 1923http:\/\/www.bolletinen."
} ]
}

```

Since we’re covering quite a lot of ground here and we’re testing your knowledge of Clojure rather than free-text search algorithms, please feel free to make the following simplifying assumptions.


* You only need to search on a single keyword. Multi-keyword searches like _Swedish IFK_ are not required
* Any document that does not have a title and abstract can be rejected
* Return empty strings if the title, abstract or url is missing from an article
* The links and sublinks in the XML document are messy and inconsistent so there is no need to index or return link data

## Installation


Clone it from [github](http://github.com/jaimeagudo/wikimediasearch)

## Usage example

`lein run` will work as usually taking the `filename` parameter `resources/enwiiki-latest-abstract.xml` as default value. To specify a different XML file use the `-f` option as


    $ lein run -f resources/small.xml
    
With .jar


	$ lein uberjar
   	$ java -jar wikimediasearch-0.1.0-standalone.jar -f resources/small.xml


* Options



````
INFO [wikimediasearch.util] - wiki cache
 index REST API.

Usage: wiki cache [options]. All the options are optional and take default values

Options:
  -f, --filename FILENAME      resources/enwiki-latest-abstract23.xml  XML input file
  -t, --thread THREAD          4                                       How many threads to compute response from request, default to 4
  -q, --queue-size QUEUE-SIZE  20000                                   Max requests queued waiting for thread pool to compute response before rejecting, 503(Service Unavailable) is returned to client if queue is full, default to 20K
  -h, --help                                                           Displays this
````

## Example request & response



	$ curl "http://localhost:8090/search?q=Kosmos"



Response

```

{"q":"Kosmos","results":[{"url":"http:\/\/en.wikipedia.org\/wiki\/Kosmos_2499","abstract":" UTC","title":"Wikipedia: Kosmos 2499"},{"url":"http:\/\/en.wikipedia.org\/wiki\/Kosmos_2221","abstract":"| launch_date = UTC","title":"Wikipedia: Kosmos 2221"},{"url":"http:\/\/en.wikipedia.org\/wiki\/List_of_Kosmos_satellites_(2501%E2%80%932750)","abstract":"The designation Kosmos ( meaning Cosmos) is a generic name given to a large number of Soviet, and subsequently Russian, satellites, the first of which was launched in 1962. Satellites given Kosmos designations include military spacecraft, failed probes to the Moon and the planets, prototypes for manned spacecraft, and scientific spacecraft.","title":"Wikipedia: List of Kosmos satellites (2501\u20132750)"},{"url":"http:\/\/en.wikipedia.org\/wiki\/Kosmos_382","abstract":"Kosmos 382 was a dummy 7K-LOK as a Soyuz 7K-L1E modification of a Soyuz 7K-L1 \"Zond\" spacecraft and was successfully test launched into Low Earth Orbit on a Proton rocket designated as (Soyuz 7K-L1E No.2) on December 2, 1970.","title":"Wikipedia: Kosmos 382"},{"url":"http:\/\/en.wikipedia.org\/wiki\/Kosmos_2501","abstract":"| spacecraft_type = Uragan-K1","title":"Wikipedia: Kosmos 2501"},{"url":"http:\/\/en.wikipedia.org\/wiki\/Dinosauria_Park","abstract":"Dinosauria Park (, O Kosmos ton Dinosauron) is a dinosaur park located near the town of Gournes in Crete, Greece, 15 km east of the city of Heraklion. The park features some fossil replicas and several animatronic dinosaur models.","title":"Wikipedia: Dinosauria Park"}]}%
```


## How to run the tests. 

`lein test` is aliased to `lein midje`, so both will work as expected


## Some performance test using AB or WRK

Server running with

* Passing these JVM Command Line Arguments or un-comment `project.clj` lines accordingly if using lein

`-Dfile.encoding=UTF-8 -Xms3G -Xmx3G -XX:+AggressiveOpts -XX:+UseCompressedOops -XX:+UnlockDiagnosticVMOptions -XX:+DebugNonSafepoints -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:+FlightRecorder -XX:StartFlightRecording=delay=60s,duration=60s,filename=myrecording2.jfr -Dclojure.compile.path=/Users/jaime/Github/wikimediasearch/target/classes -Dwikimediasearch.version=0.0.1-SNAPSHOT -Dclojure.debug=false`

* Passing `-t 200` option and default `queue-size`


Using [ApacheBench](https://httpd.apache.org/docs/2.2/programs/ab.html). Included on Mac OS (nothing to install).  On Debian `sudo apt-get install apache2-utils`

```
    $ cd benchmark
    $ ./test_ab.sh
```



###Typical output on my laptop (Mac i5 2,6GHz)

```
$ cd benchmark
$ ./test_ab.sh                                                                                  [±master ●●]
This is ApacheBench, Version 2.3 <$Revision: 1604373 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 10000 requests
Completed 20000 requests
Completed 30000 requests
Completed 40000 requests
Completed 50000 requests
Completed 60000 requests
Completed 70000 requests
Completed 80000 requests
Completed 90000 requests
Completed 100000 requests
Finished 100000 requests


Server Software:        http-kit
Server Hostname:        localhost
Server Port:            8090

Document Path:          /search?q=Kosmos
Document Length:        1560 bytes

Concurrency Level:      10
Time taken for tests:   32.058 seconds
Complete requests:      100000
Failed requests:        0
Keep-Alive requests:    100000
Total transferred:      191400000 bytes
HTML transferred:       156000000 bytes
Requests per second:    3119.30 [#/sec] (mean)
Time per request:       3.206 [ms] (mean)
Time per request:       0.321 [ms] (mean, across all concurrent requests)
Transfer rate:          5830.41 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       2
Processing:     1    3   2.0      3      74
Waiting:        0    3   2.0      3      74
Total:          1    3   2.0      3      74

Percentage of the requests served within a certain time (ms)
  50%      3
  66%      3
  75%      4
  80%      4
  90%      5
  95%      5
  98%      7
  99%      8
 100%     74 (longest request)

```

The performance looks good enough dealing with over 3000 request/second and having a quick look at the [flight record](./benchmark/flight_recording_18045Clojure42957.jfr) there aren't any obvious bottlenecks, we don't even need more than 1,8GB of RAM ![RAM](./benchmark/img/mem.png) . The *hottest* method takes 2.62% of the execution time and it's `java.lang.Thread.run` which suggest that the biggest task is the coordination of the threads rather than the internal logic. 
![method](./benchmark/img/methods.png)



* Alternatively you can observe similar results using [wrk](https://github.com/wg/wrk). On Mac OS you can install it easily with `brew install wrk`. For Linux refer to the github repo

```
	$ cd benchmark   
	$ ./test_wrk.sh
```


## License

Copyright © 2015 Jaime Agudo

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
