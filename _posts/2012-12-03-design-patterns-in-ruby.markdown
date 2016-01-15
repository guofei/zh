---
layout: post
title: "Design Patterns In Ruby"
date: 2012-12-03 19:11
comments: true
categories: [ruby,design pattern,book]
---
### Template Method

{% highlight ruby %}
class Report
  def initialize
    @title = 'Monthly Report'
    @text =  ['Things are going', 'really, really well.']
  end

  def output_report
    output_start
    output_head
    output_body_start
    @text.each do |line|
      output_line(line)
    end
    output_body_end
    output_end
  end

  def output_start
  end

  def output_head
    output_line(@title)
  end

  def output_body_start
  end

  def output_line(line)
    raise 'Called abstract method: output_line'
  end

  def output_body_end
  end

  def output_end
  end
end

class PlainTextReport < Report
  def output_line(line)
    puts(line)
  end
end

class HTMLReport < Report
  def output_start
    puts('<html>')
  end
  def output_head
    puts('  <head>')
    puts("    <title>#{@title}</title>")
    puts('  </head>')
  end
  def output_body_start
    puts('<body>')
  end
  def output_line(line)
    puts("  <p>#{line}</p>")
  end
  def output_body_end
    puts('</body>')
  end
  def output_end
    puts('</html>')
  end
end
{% endhighlight %}

### Strategy
{% highlight ruby %}
# 1
class Report
  attr_reader :title, :text
  attr_accessor :formatter

  def initialize(formatter)
    @title = 'Monthly Report'
    @text =  ['Things are going', 'really, really well.']
    @formatter = formatter
  end

  def output_report
    @formatter.output_report(self)
  end
end

class HTMLFormatter
  def output_report(context)
    puts('<html>')
    puts('  <head>')
    # Output The rest of the report ...

    puts("    <title>#{context.title}</title>")
    puts('  </head>')
    puts('  <body>')
    context.text.each do |line|
      puts("    <p>#{line}</p>")
    end
    puts('  </body>')

    puts('</html>')
  end
end

class PlainTextFormatter
  def output_report(context)
    puts("***** #{context.title} *****")
    context.text.each do |line|
      puts(line)
    end
  end
end

# 2
class Report
  attr_reader :title, :text
  attr_accessor :formatter

  def initialize(&formatter)
    @title = 'Monthly Report'
    @text =  [ 'Things are going', 'really, really well.' ]
    @formatter = formatter
  end

  def output_report
    @formatter.call( self )
  end
end

HTML_FORMATTER = lambda do |context|
  puts('<html>')
  puts('  <head>')
  puts("    <title>#{context.title}</title>")
  puts('  </head>')
  puts('  <body>')
  context.text.each do |line|
    puts("    <p>#{line}</p>" )
  end
  puts('  </body>')
  puts('</html>')
end

PLAIN_FORMATTER = lambda do |context|
  puts("***** #{context.title} *****")
  context.text.each do |line|
    puts(line)
  end
end
{% endhighlight %}

### Observer
{% highlight ruby %}
class Payroll
  def update( changed_employee )
    puts "Cut a new check for #{changed_employee.name}!"
    puts "His salary is now #{changed_employee.salary}!"
  end
end

module Subject
  def initialize
    @observers=[]
  end

  def add_observer(observer)
    @observers << observer
  end

  def delete_observer(observer)
    @observers.delete(observer)
  end

  def notify_observers
    @observers.each do |observer|
      observer.update(self)
    end
  end
end

class Employee
  include Subject

  attr_reader :name, :address
  attr_reader :salary

  def initialize( name, title, salary)
   super()
   @name = name
   @title = title
   @salary = salary
  end

  def salary=(new_salary)
    @salary = new_salary
    notify_observers
  end
end

# library
class Ticker          ### Periodically fetch a stock price.
  include Observable

  def initialize(symbol)
    @symbol = symbol
  end

  def run
    lastPrice = nil
    loop do
      price = Price.fetch(@symbol)
      print "Current price: #{price}\n"
      if price != lastPrice
        changed                 # notify observers
        lastPrice = price
        notify_observers(Time.now, price)
      end
      sleep 1
    end
  end
end

class Price           ### A mock class to fetch a stock price (60 - 140).
  def Price.fetch(symbol)
    60 + rand(80)
  end
end

class Warner          ### An abstract observer of Ticker objects.
  def initialize(ticker, limit)
    @limit = limit
    ticker.add_observer(self)
  end
end

class WarnLow < Warner
  def update(time, price)       # callback for observer
    if price < @limit
      print "--- #{time.to_s}: Price below #@limit: #{price}\n"
    end
  end
end

class WarnHigh < Warner
  def update(time, price)       # callback for observer
    if price > @limit
      print "+++ #{time.to_s}: Price above #@limit: #{price}\n"
    end
  end
end

ticker = Ticker.new("MSFT")
WarnLow.new(ticker, 80)
WarnHigh.new(ticker, 120)
ticker.run
{% endhighlight %}

### Composite
{% highlight ruby %}
class Task
  attr_accessor :name
  def initialize(name)
    @name = name;
  end

  def get_time_required
    0.0
  end
end

class AddDryIngredientsTask < Task
  def initialize
    super('Add dry ingredients')
  end

  def get_time_required
    1.0
  end
end

class MixTask < Task
  def initialize
    super('Mix that batter up!')
  end

  def get_time_required
    3.0
  end
end

class AddLiquidsTask < Task
  def initialize
    super('Add Liquids task')
  end

  def get_time_required
    5.0
  end
end

class CompositeTask < Task
  def initialize(name)
    super(name)
    @sub_tasks = []
  end

  def add_sub_task(task)
    @sub_tasks << task
    task.parent_task = self
  end

  def remove_sub_task(task)
    @sub_tasks.delete(task)
  end

  def get_time_required
    time = 0.0
    @sub_tasks.each {|task| time += task.get_time_required}
    time
  end
end

class MakeBatterTask < CompositeTask
  def initialize
    super('Make batter')
    add_sub_task(AddDryIngredientsTask.new)
    add_sub_task(MixTask.new)
    add_sub_task(AddLiquidsTask.new)
  end
end

task = MakeBatterTask.new
puts task.name
puts task.get_time_required
{% endhighlight %}

### Iterator

### Command
{% highlight ruby %}
require "fileutils"

class Command
  attr_reader :description
  def initialize(description)
    @description = description
  end

  def execute
  end
end

class CreateFile < Command
  def initialize(path, contents)
    super("Create file : #{path}")
    @path = path
    @contents = contents
  end

  def execute
    f = File.open(@path, "w")
    f.write(@contents)
    f.close
  end

  def unexecute
    File.delete(@path)
  end
end

class DeleteFile < Command
  def initialize(path)
    super("Delete file : #{path}")
    @path = path
  end

  def execute
    if(File.exists?(@path))
      @contents = File.read(@path)
    end
    File.delete(@path)
  end

  def unexecute
    f = File.open(@path, "w")
    f.write(@contents)
    f.close
  end
end

class CopyFile < Command
  def initialize(source, target)
    super("Copy file : #{source} to #{target}")
    @source = source
    @target = target
  end

  def execute
    if(File.exists?(@target))
      @contents = File.read(@target)
    end
    FileUtils.copy(@source, @target)
  end

  def unexecute
    File.delete(@target)
    if(@contents)
      f = File.open(@target, "w")
      f.write(@contents)
      f.close
    end
  end
end

class CompositCommand < Command
  def initialize
    @commands = []
  end

  def add_command(cmd)
    @commands << cmd
  end

  def execute
    @commands.each { |cmd| cmd.execute }
  end

  def unexecute
    @commands.reverse.each { |cmd| cmd.unexecute }
  end

  def description
    description = ""
    @commands.each { |cmd| description += cmd.description + "\n"}
    description
  end
end

cmds = CompositCommand.new

cmds.add_command(CreateFile.new("file1.txt", "hello world10\n"))
cmds.add_command(CopyFile.new("file1.txt", "file2.txt"))
cmds.add_command(DeleteFile.new("file1.txt"))

cmds.execute
cmds.unexecute
{% endhighlight %}

### Adapter
{% highlight ruby %}
class LegacyTextObject
  def print
    show_message
  end
end

lto = LegacyTextObject.new
class << lto
  def print
    show_message
  end
end
{% endhighlight %}

### Proxy
{% highlight ruby %}
class BankAccount
  attr_reader :balance
  def initialize(balance)
    @balance = balance
  end

  def deposit(amount)
    @balance += amount
  end

  def withdraw(amount)
    @balance -= amount
  end
end

class BankAccountProxy
  def initialize(real_object, owner_name)
    @real_object = real_object
    @owner_name = owner_name
  end

  def balance
    check_access
    @real_object.balance
  end

  def deposit(amount)
    check_access
    @real_object.deposit(amount)
  end

  def withdraw(amount)
    check_access
    @real_object.withdraw(amount)
  end

  def check_access
    if(Etc.getlogin != @owner_name)
      raise "Illegal access: #{@owner_name} cannot access account."
    end
  end
end

account = BankAccount.new(100)
proxy = BankAccountProxy.new(account, "murayama")
puts proxy.deposit(50)
puts proxy.withdraw(10)

class VirtualAccountProxy
  def initialize(starting_balance)
    @starting_balance = starting_balance
  end

  def balance
    subject.balance
  end

  def deposit(amount)
    subject.deposit(amount)
  end

  def withdraw(amount)
    subject.withdraw(amount)
  end

  def subject
    @subject || (@subject = BankAccount.new(@starting_balance))
  end
end

# method_missing
class BankAccountProxy
  def initialize(real_object, owner_name)
    @real_object = real_object
    @owner_name = owner_name
  end

  def method_missing(name, *args)
    check_access
    @real_object.send(name, *args)
  end

  def check_access
    if(Etc.getlogin != @owner_name)
      raise "Illegal access: #{@owner_name} cannot access account."
    end
  end
end
{% endhighlight %}

### Decorator
{% highlight ruby %}
class SimpleWriter
  def initialize(path)
    @file = File.open(path, "w")
  end

  def write_line(line)
    @file.print(line)
    @file.print("\n")
  end

  def pos
    @file.pos
  end

  def rewind
    @file.rewind
  end

  def close
    @file.close
  end
end

require "forwardable"
class WriterDecorator
  extend Forwardable

  def_delegators :@real_writer, :write_line, :pos, :rewind, :close

  def initialize(real_writer)
    @real_writer = real_writer
  end
end

class NumberingWriter < WriterDecorator

  def initialize(real_writer)
    super(real_writer)
    @line_number = 1
  end

  def write_line(line)
    @real_writer.write_line("#{@line_number} : #{line}")
  end
end

f = NumberingWriter.new(SimpleWriter.new("file1.txt"))
f.write_line("Hello world.")
f.close

class TimestampingWriter < WriterDecorator
  def write_line(line)
    @real_writer.write_line("#{Time.new} : #{line}")
  end
end

f = TimestampingWriter.new(SimpleWriter.new("file1.txt"))
f.write_line("Hello world.")
f.close

f = TimestampingWriter.new(NumberingWriter.new(SimpleWriter.new("file1.txt")))
f.write_line("Hello world.")
f.close

#
f = SimpleWriter.new("file1.txt")
class << f
  alias old_write_line write_line

  def write_line(line)
    old_write_line("#{Time.new} : #{line}")
  end
end

# module
module NumberingWriter
  attr_reader :line_number

  def write_line(line)
    @line_number = 1 unless @line_number
    super("#{@line_number} : #{line}")
    @line_number += 1
  end
end

module TimestampingWriter
  def write_line(line)
    super("#{Time.new} : #{line}")
  end
end

f = SimpleWriter.new("file1.txt")
f.extend(NumberingWriter)
f.extend(TimestampingWriter)
f.write_line("Hello world3.")
f.close
{% endhighlight %}

### Singleton
### Factory Method
{% highlight ruby %}
class Duck
  def initialize(name)
    @name = name
  end

  def speak
    puts("Duck #{@name} speak.")
  end
end

class Frog
  def initialize(name)
    @name = name
  end

  def speak
    puts("Frog #{@name} speak.")
  end
end

class Pond
  def initialize(number_animals)
    @animals = []
    number_animals.times do |i|
      animal = new_animal("Animal#{i}")
      @animals << animal
    end
  end

  def simulate_one_day
    @animals.each { |animal| animal.speak}
  end
end

class DuckPond < Pond
  def new_animal(name)
    Duck.new(name)
  end
end

class FrogPond < Pond
  def new_animal(name)
    Frog.new(name)
  end
end
{% endhighlight %}

### Abstract Factory
{% highlight ruby %}
class Algae
  def initialize(name)
    @name = name
  end

  def grow
    puts "Algae #{@name} grow."
  end
end

class WaterLily
  def initialize(name)
    @name = name
  end

  def grow
    puts "WaterLily #{@name} grow."
  end
end

class Pond
  def initialize(number_animals, number_plants)
    @animals = []
    number_animals.times do |i|
      animal = new_animal("Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant = new_plant("Plant#{i}")
      @plants << plant
    end
  end

  def simulate_one_day
    @plants.each { |plant| plant.grow}
    @animals.each { |animal| animal.speak}
  end
end

class DuckWaterLilyPond < Pond
  def new_animal(name)
    Duck.new(name)
  end

  def new_plant(name)
    WaterLily.new(name)
  end
end

class FrogAlgaePond < Pond
  def new_animal(name)
    Frog.new(name)
  end

  def new_plant(name)
    Algae.new(name)
  end
end

# class object
class Pond
  def initialize(number_animals, animal_class, number_plants, plant_class)
    @animals = []
    number_animals.times do |i|
      animal = animal_class.new("Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant = plant_class.new("Plant#{i}")
      @plants << plant
    end
  end

  def simulate_one_day
    @plants.each { |plant| plant.grow}
    @animals.each { |animal| animal.speak}
  end
end

Pond.new(3, Duck, 2, Algae).simulate_one_day
{% endhighlight %}

### Builder
### Interpreter

### Reference
[blog](http://murayama.hatenablog.com/category/%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)
