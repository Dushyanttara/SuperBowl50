# SuperBowl50
Read NFL game data into Spark and try to predict the over/under for the big game

### Processing the data yourself (optional)...



### Launching Spark shell & loading the data


### Exploring the data


### Running the KNN algorithm
Note: you need these statistics to standardize some of the features:<br>
<b>val tempStats = rawFeatures.filter(m=> m._5 > -99.0).map{ p => (p._5) }.stats<br>
val windStats = rawFeatures.filter(m=> m._7 > -99.0).map{ p => (p._7) }.stats<br>
val spreadStats = rawFeatures.filter(m=> m._8 > -99.0).map{ p => (p._8) }.stats<br>
val overStats = rawFeatures.map{ p => (p._9) }.stats<br>
</b><br>
Note: edit this string to change the forecast for the Superbowl<br>
<b>val superString = "Super Bowl 50 - February 7th, 2016 |6:30pm|outdoors|grass |60 degrees relative humidity 72%, wind 10 mph|4.5|45.0 |00|2015"<br>
var parsedSuper = parseData(superString.split('|').map(elem => elem.trim))<br>
</b><br>
Use these shell commands to calculate the 50 most-similar games:<br>
<b>def standardize(measure:Double,stats:org.apache.spark.util.StatCounter):Double = {<br>
   ((measure - stats.min) / (stats.max - stats.min))<br>
}<br>
<br>
def calcDistance(game:(String, Double, Double, Double, Double, Double, Double, Double, Double, Double, String),
        test:(String, Double, Double, Double, Double, Double, Double, Double, Double, Double,String)):(Double) = {<br>
   var distance = 0.0<br>
   //(parts(0),time,roof,field,temperature,humidity,wind,spread,overunder,points)<br>
   distance += math.pow(game._2-test._2,2)<br>
   distance += math.pow(game._3-test._3,2)<br>
   distance += math.pow(game._4-test._4,2)<br>
   distance += math.pow(standardize(game._5,tempStats)-standardize(test._5,tempStats),2)<br>
   distance += math.pow(game._6/100-test._6/100,2)<br>
   distance += math.pow(standardize(game._7,windStats)-standardize(test._7,windStats),2)<br>
   distance += math.pow(standardize(game._8,spreadStats)-standardize(test._8,spreadStats),2)<br>
   distance += math.pow(standardize(game._9,overStats)-standardize(test._9,overStats),2)<br>
   (distance)<br>
}<br>
val distances = rawFeatures.filter(m=> m._5 > -99.0 && m._6> -99.0 && m._7> -99.0 && m._8> -99.0 && m._11.toDouble >2001).map{ game =><br>
   (game._1,calcDistance(game,parsedSuper),game._10)<br>
}<br>
distances.sortBy(_._2).take(100).foreach(println)<br>
</b><br>
### Creating the graphs in R Studio
Note: These are the R commands I used to create the plot of points by season. It assumes your season data is located in a file called <b>stats</b><br>
<b>library("colorspace")<br>
seasons <- read.csv("stats",stringsAsFactors=FALSE,header=FALSE)<br>
colnames(seasons) <- c("season","overunder","points")<br>
attach(seasons)<br>

pcol <- c("red","blue","forestgreen")<br>
yl=c(30,50)<br>
plot(season,overunder,ylim=yl,main="NFL Scoring by Season", xlab="Seasons", ylab="Points per Game",
  col=pcol[1],pch=20,cex=1.5,type='b')<br>
  points(season,points,col=pcol[2],pch=20,cex=1.5,type='b')<br>
  legend(2005,35,c("avg. over/under","avg. actual"),pch=c(20,20),col=c(pcol[1],pcol[2]))<br>
</b><br>

Note: These are the R commands I used to create the histogram of points in similar games. It assumes your nearest neighbor output is located in a file called <b>knn</b><br>
<b>neighbors <- read.csv("knn",stringsAsFactors=FALSE,header=FALSE)<br>
colnames(neighbors) <- c("game","year","distance","points")<br>
attach(neighbors)<br>
medianPoints <- median(points)<br>
hist(points, main="Distribution of Points in Similar Games since 2002",xlab="Points Scored",ylab="Number of Games", 
   ylim=c(0,20),breaks=20)<br>
abline(v=c(45),lty=2,lwd=8,col="red")<br>
abline(v=c(medianPoints),lty=2,lwd=8,col="green")<br>
legend(10,17,c("SuperBowl Over/Under","Median Points, Similar Games"),pch=c(20,20),col=c("red","green"))<br>
