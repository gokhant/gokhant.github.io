---
layout: post
title:  "50-Batch"
date:   2016-02-28 15:02:05 +0200
categories: 
- programming
- java
- springframework
- batch

tags:
- spring
- batch
- java
- csv
- gsm
- telecommunication
- cell
- basestation
- timing advance
- propagation delay

---
{:.center-image}
![Welcome](/images/50/welcome_to_batch.jpg)

{:.credit}
Photo Credit: [Rachel Kramer Bussel (Last visit to Batch in West Village)](https://www.flickr.com/photos/rkbcupcakes/albums/72157615800966218)

Of course this is not a cafe called *Batch*, only *Spring Batch* is here and there are no cupcakes nor espressos here, sorry for that.
\\
\\
If you've come this far, you have already read [51-Do you love Spring or hate it?][link-51], if not please do so.
In order to continue, ***knowing*** basic [Spring](http://www.spring.io), [some annotations](https://dzone.com/refcardz/spring-annotations) and [Apache Maven](http://maven.apache.org) is required, only then you may get the best of this post.
\\
\\
As I mentioned [earlier][link-51], I need to process CSV files with sizes varying from 180 to 500 MBs. These files contain base stations and number of subscribers according to their 
distances to the base station (*I may sometimes use **cell** from now on*) on an hourly basis. The aim is finding the area with the greatest number of subscribers by using these files.
At this point let's talk about some GSM concepts very superficially:

> *Distances* are calculated according to [timing advance (TA)](https://en.wikipedia.org/wiki/Timing_advance) or [propagation delay (PD)](https://en.wikipedia.org/wiki/Propagation_delay). 
Both of them are very briefly the length of time a signal takes to travel between the base station and the mobile phone. Let's also add that TA is for 2G and PD is 3G.
\\
\\
> Let's assume that blue marker 0 below identifies our 3G serving cell's location. This cell has a beamwidth[^beamwidth] of 67<sup>o</sup> and the arc drawn in navy blue has a radius of 234 m. The people in this arc mostly have a PD of 0. 
> Each PD value denotes a distance of 234 m; such as subscribers with PD 1 will be mostly between 234m and 468m to the serving cell. In order to draw this, azimuth of the cell is also required, which is 60<sup>o</sup> by the way; 
but I will tell about azimuth and beamwidth later, in another post.
\\
\\
> For more details you can check [this article](http://www.telecomhall.com/analyzing-coverage-with-propagation-delay-pd-and-timing-advance-ta-gsm-wcdma-lte.aspx).

> {:.center-image}
> ![ZZ](/images/50/measurements.png)
> 
> {:.credit}
> The location of the 3G cell above is completely fictional.



Let's return back to the csv files, below is a sample for 3G. As you can see, there is one line for every hour for a cell, meaning each cell contains 24 lines of data which should be accumulated before being used.
Imagine the size of the file with more than 100K cells! 

~~~~
cell_id,data_time,pd0,pd1,pd2,pd3,pd4,...
15,23.02.2016 00:00:00,1200,2400,400,300,142...
15,23.02.2016 01:00:00,1000,2000,345,300,112...
15,23.02.2016 02:00:00,900,1700,245,300,92...
...
15,23.02.2016 23:00:00,1800,3500,487,400,162...
16,23.02.2016 00:00:00,800,2500,457,100,38...
...
~~~~

For starters, here is the *pom.xml*, using not only Spring Batch but also Spring Boot which enables to code easier and faster after some learning curve. 

~~~~ xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>gokhan.java.spring.batch</groupId>
	<artifactId>example</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Batch Example</name>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.3.1.RELEASE</version>
		<!-- <relativePath/>  lookup parent from repository -->
	</parent>


	<repositories>
		<repository>
			<id>spring-releases</id>
			<url>http://repo.spring.io/release</url>
			<snapshots><enabled>true</enabled></snapshots>
		</repository>
	</repositories>

    <pluginRepositories>
		<pluginRepository>
			<id>spring-releases</id>
			<url>http://repo.spring.io/release</url>
			<snapshots><enabled>true</enabled></snapshots>
		</pluginRepository>
    </pluginRepositories>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-batch</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
</project>
~~~~

Initial source codes will be here very shortly in my github account, I will update this post, stay tuned!


- - - - - -

[^beamwidth]: It is the angle between the edges (blue lines) of seving cell's signal

[link-51]: {% post_url 2016-02-20-do-you-love-spring-or-hate-it %}