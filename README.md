#Dynamic finder methods
>Aim of this article is to explain meta programming usage of gem mongoid dynamic finder.


There are many ways to use mongodb with rails, one of them is using Gem called [Mongoid]. Which will give you activerecord like usage models.

```sh
class Artist
  include Mongoid::Document
  field :name, type: String
  embeds_many :instruments
end

syd = Artist.where(name: "Syd Vicious").between(age: 18..25).first
```
if you would like to use activerecord like dynamic finders then you ll have to install gem called [Mongoid dynamic finder].
Just add the gem and bundle.

```sh
Artist.find_by_name_and_title('Some Artist' , 'Mr.')
```
So how does it work, today i ll explain how this dyanmic finders work by explaining mongoid dynamic finders gem.

-----------

##Where magic happens

```sh
def method_missing(method_id, *args, &block)
      conditions = {}
      bang = false
```
Calling find_by_name method will end up here method missing, because there is not a method called by that name. Method_missing method initiates conditions hash and bang variable. method_id will be find_by_name and arguments will be 'Some Artist', 'Mr.'

```sh
1      case method_id.to_s
2      when /^find_(all|last||first)_?by_([_a-zA-Z]\w*)(!?)$/
3        finder_type = $1.blank? ? :first : $1.to_sym
4        bang = true if $3 == '!'
5
6        $2.split(/_and_/).each_with_index do |attr, i|
7          conditions[attr] = args[i]
8        end
```
 - line 1 turns method name to string.
 - if method_id is acceptable by regular expression on line 2
than regular expression parses 3 groups of data from method_id 
finder_type symbol variable will be :first if it is called just find
or find_first, if it is called find_all or find_last then finder_type will be simply :all or :last.
 - if there is bang at the end of method call then bang variable will be set to true, on line 4.
 - In order to find which conditions for find method; second part of parsing $2 will be split into array. So if it is find_by_name_and_title
then conditions hash will have keys name and title , their values will be the order of them.

```sh
        result = find(finder_type, :conditions => conditions)
```
Here is the simple part , once finder_type and conditions information is parsed then calls find method on [Mongoid] gem and saves returned information to result variable.

------

###Conculusion

I tried to explain usage of meta programming at [Mongoid dynamic finder] gem, i think it is a simple example of using method missing. 

Activerecords gem has also dynamic finders, and its very complicated source code to dive in. Thats why i prefered this simple example. 

----

**Article by [M.Cenk Ayberkin]**

[Mongoid dynamic finder]:https://github.com/mitijain123/mongoid_dynamic_finder
[Mongoid]:http://mongoid.org/en/mongoid/index.html
[M.Cenk Ayberkin]:https://github.com/cenkayberkin
