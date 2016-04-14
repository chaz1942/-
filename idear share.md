#观点分享
##1.在使用ransack出现的问题
###问题是？
>网站提供的关于ransack的搜索条件介绍的不是很详细，只介绍了_cont和_start条件，实际中需求多种搜索条件
>例如相等条件，不等于关系的条件，没有很详细的关于每种条件的说明。
### 如何解决
>首先google了一下，关于这个资料也不是很多
>然后下载了一下源码，先大致浏览了一下源码，先从ransack函数搜索，然后搜索了一下“cont”，找到了一个文件储存了所有的字段()

```
file from ransack/lib/ransack/locale/zh-CN.yml
      eq: "等于"
      eq_any: "等于任意值"
      eq_all: "等于所有值"
      not_eq: "不等于"
      not_eq_any: "不等于任意值"
      not_eq_all: "不等于所有值"
      matches: "符合"
      matches_any: "符合任意条件"
      matches_all: "符合所有条件"
      does_not_match: "不符合"
      does_not_match_any: "符合任意条件"
      does_not_match_all: "不符合所有条件"
      lt: "小于"
      lt_any: "小于任意一个值"
      lt_all: "小于所有值"
      lteq: "小于等于"
      lteq_any: "小于等于任意一个值"
      lteq_all: "小于等于所有值"
      gt: "大于"
      gt_any: "大于任意一个值"
      gt_all: "大于所有值"
      gteq: "大于等于"
      gteq_any: "大于等于任意一个值"
      gteq_all: "大于等于所有值"
      in: "被包含"
      in_any: "被任意值包含"
      in_all: "被所有值包含"
      not_in: "不被包含"
      not_in_any: "不被任意值包含"
      not_in_all: "不被所有值包含"
      cont: "包含"
      cont_any: "包含任意一个值"
      cont_all: "包含所有值"
      not_cont: "不包含"
      not_cont_any: "不包含任意一个值"
      not_cont_all: "不包含所有值"
      start: "以改值开始"
      start_any: "以任意一个值开始"
      start_all: "以所有值开始"
      not_start: "不以改值开始"
      not_start_any: "不以任意一个值开始"
      not_start_all: "不以所有值开始"
      end: "以改值结尾"
      end_any: "以任意一个值结尾"
      end_all: "以所有值结尾"
      not_end: "不以改值结尾"
      not_end_any: "不以任意一个值结尾"
      not_end_all: "不以所有值结尾"
      'true': "等于true"
      'false': "等于false"
      present: "有值"
      blank: "为空"
      'null': "是null"
      not_null: "不是null"

```

>这个文件应该是生成文档的时候适应多语言环境下使用的，里面恰好包含了所有搜索条件以及对搜素条件的叙述。

###对这个问题的解决思路
>1.以后如果有类似的问题，从源码中学习用法是个比较快的捷径

>2.参考某个gem包中的用法，或者下载这个gem包的demo然后生成文档，查看文档。

#2.一个有意思的数学模型（选自R和RUBY数据分享之旅第三章）
## 提出问题
>探讨员工数量和卫生间数量关系，即如何根据员工数量决定设定卫生间的数量情况？？？

>首先进行假设，在一定的员工数（假设为70）设卫生间的数量，达到卫生间数量最少，并使等候的人数最少，根据查阅相关资料，得知英国健康与安全执行局（HSE）对卫生间的数量建议如下

| 员工数量       | 卫生间数量  	  | 小便器数|
| :-----------: |:-------------:| :-----:|
|       1-15   	|   	1  	 	|    	1	|
|     16-30     	|     	2 		|     	1  	|
|     31-45		|		2		|		2	|
|		46-60		|		3		|		2	|

>但是HSE的规定究竟和不合理，我们需要做一番模拟来研究这种情况。

## 描述并建立模型

```
step1.设定模型描述员工是怎么样使用卫生间的
	在任何一个时间上，办公室的人都可以进入卫生间。某员工进入卫生间通常不会影响其他员工是非进入卫生间。因此简单来看，办公室中卫生间使用情况可以使用泊松过程来建模。这个过程是一种随机过程，事件会随时间连续的发生，时间之间相互独立。
	此外我们需要找出，一个人平均要上多少次厕所，通过一番资料查找，成年人八次，平均每三小时一次。通情况下，工作时间从8:30到下午5:30，一共9个小时。因此，员工每人每天要使用三次办公室的卫生间。
	我们把员工上厕所的模型描述为：
	员工在办公室的9个小时之间（540分钟），每隔一分钟做一次决定是否要上厕所，而每次决定去的概率为3/540.关于如厕行为，我们设定为每人一分钟。
	有了模型的描述，接下来尝试通过编程模拟卫生间的使用情况。
```

>Restroom类


	class Restroom
  		attr_reader :queue # a queue of people waiting to enter the restroom
  		attr_reader :facilities

  		def initialize(facilities_per_restroom=3)
    		@queue = []
    		@facilities = [] # the facilties in this restroom
    		facilities_per_restroom.times { @facilities << Facility.new }    
  		end

  		def enter(person)
    		unoccupied_facility = @facilities.find { |facility| not facility.occupied?}
    		if unoccupied_facility
      			unoccupied_facility.occupy person
    		else
      			@queue << person
      			Person.population.delete person
    		end
  		end

  		def tick
    		@facilities.each { |f| f.tick }
  		end

	end

>这个类代表了卫生间，它有两个可读属性：等待使用队列的队列长度，以及卫生间内的一些类设备。
>该类有两个方法：

>* enter，它寻找下一个可用设备，并占用，如果没有闲置设备就将员工置于等待队列中。

>* tick， 他将逐一调用各个设备的tick方法，也就是员工如厕行为的操作

>Facility类

``` 
class Facility
  def initialize
    @occupier = nil # no one is occupying this facility at the start
    @duration = 0 # how long the facility has been occupied
  end

  def occupy(person)
    unless occupied? # if this facility is occupied return false
      @occupier = person # this facility is occupied!
      @duration = 1 # this facility has been occupied for 1 tick
      Person.population.delete person # remove the person from the population since he's in the queue now
      true
    else
      false
    end
  end

  def occupied?
    not @occupier.nil?
  end

  def vacate
    Person.population << @occupier # put the person back into the population
    @occupier = nil
  end

  def tick    
    if occupied? and @duration > @occupier.use_duration
      vacate
      @duration = 0
    elsif occupied?
      @duration += 1 
    end
  end  
end

```
>该类代表卫生间小便池，有四个方法:

>* occupier是由Restroom中调用的。根据设备的占用返回true或者false
>* occupied？检查是否被占用
>* vacate 将员工移除
>* tick Restroom类调用的

>Person类

```
class Person
  @@population = []
  attr_reader :use_duration 
  attr_accessor :frequency
  
  def initialize(frequency=4,use_duration=1)
    @frequency = frequency # how many times the person uses the facility over the experiment period
    @use_duration = use_duration  # how long this person occupies the facility
  end    

  def self.population
    @@population
  end

  def need_to_go?
    rand(DURATION) + 1 <= @frequency
  end
end

```
>属性:

>* population 储存建模过程中所有人群
>* use_duration 描述一个人多长时间上完厕所
>* frequency 描述一个人使用频率
	
	step2.运行模拟
	1.进行模拟，对数据进行暂时存储
	2.将数据写到csv文件中，见代码和运行结果。
	
>模拟脚本如下:

```
require 'csv'
require './restroom'

# Simulation script 1 

frequency = 3 # how many times a person goes to the restroom with the period
facilities_per_restroom = 3
use_duration = 1
population_range = 10..600 # using a population range of 0 to 500 people

data = {}
population_range.step(10).each do |population_size|
  Person.population.clear
  population_size.times { Person.population << Person.new(frequency, use_duration) } # create the population
  data[population_size] = [] #initialize the temp data store
  restroom = Restroom.new facilities_per_restroom # create the restroom    

  # iterate over a period
  DURATION.times do |t|
    data[population_size] << restroom.queue.size # we want the queue size
    queue = restroom.queue.clone # create a temporary queue so that we can sort people between the facilities and the restroom queue for this "tick"
    restroom.queue.clear # clear the queue to prepare for the sorting
    
    # take each person from the temporary queue and try adding them to a facility
    until queue.empty? 
      restroom.enter queue.shift # de-queue the person at the front of the line, place in an unoccupied facility or, if none, back to the restroom queue
    end
    

    # for each person in the population check if he needs to go
    Person.population.each do |person|
      if person.need_to_go?
        restroom.enter person
      end
    end
    restroom.tick
  end
end

# write the temp store into CSV
CSV.open('simulation1.csv', 'w') do |csv|
  # setup labels
  lbl = []
  population_range.step(10).each {|population_size| lbl << population_size  }
  csv << lbl  
  
  # write the data
  DURATION.times do |t|
    row = []
    population_range.step(10).each do |population_size|
      row << data[population_size][t]
    end
    csv << row
  end
end

```

![result1](http://cl.ly/ffSr/figure3-2.png)

	step3.对模型修正再次模拟
	1.我们刚才的运行说明了，卫生间设备数量固定，而在连续的9个小时内改变办公室的总人数。它回答了，在3个卫生间设备，可以满足多少人的需求。但是我们这个建模的出发点是探究在人数确定的情况下建设多少卫生间设备。于是我们把上一个问题反过来，固定人数，变动卫生间的设备数。
	2.再次运行模拟
	
>模拟脚本

```
require 'csv'
require './restroom'

# Simulation script 2

frequency = 3 # how many times a person goes to the restroom with the period
use_duration = 1
population_size = 1000 # using a population range of 1000 people
facilities_per_restroom_range = 1..30
data = {}
facilities_per_restroom_range.each do |facilities_per_restroom|
  Person.population.clear
  population_size.times { Person.population << Person.new(frequency, use_duration) } # create the population
  data[facilities_per_restroom] = [] #initialize the temp data store
  restroom = Restroom.new facilities_per_restroom # create the restroom    

  # iterate over a period
  DURATION.times do |t|
    queue = restroom.queue.clone # clone the queue so that we don't mess up the live one
    restroom.queue.clear # clear the queue
    data[facilities_per_restroom] << queue.size # we want the queue size
    
    # let everyone from the queue enter the restroom first
    until queue.empty? 
      restroom.enter queue.shift # de-queue the first person in line and move him to the restroom
    end

    # for each person in the population check if he needs to go
    Person.population.each do |person|
      if person.need_to_go?
        restroom.enter person
      end
    end
    restroom.tick
  end
end

# write the temp store into CSV
CSV.open('simulation2.csv', 'w') do |csv|
  # setup labels
  lbl = []
  facilities_per_restroom_range.each {|facilities_per_restroom| lbl << facilities_per_restroom  }
  csv << lbl  
  
  # write the data
  DURATION.times do |t|
    row = []
    facilities_per_restroom_range.each do |facilities_per_restroom|
      row << data[facilities_per_restroom][t]
    end
    csv << row
  end
end

```
	
![result2](http://f.cl.ly/items/2O2d331j2X2O0m2U3N2d/figure3-5.png)

>关于边际效应
>>经济学名词，指消费者在每增加一个单位消费品的时候，其产生的效用成递减速趋势。

>>由图标可以看到将横坐标切分为一个一个的小块Δx，每个Δx对应Δy的变化是随着x的增加而减少，上图显示的关系时随人数变化，等待队列长度的情况。
