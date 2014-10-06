#Dynamic finder methods

>The aim of this article is to explain the meta programming usage of the gem [Mongoid dynamic finder]


There are many ways to use mongodb with rails, one of them is using a gem called [Mongoid] which will give you activerecord like usage models.

```sh
class Artist
  include Mongoid::Document
  field :name, type: String
  embeds_many :instruments
end

syd = Artist.where(name: "Syd Vicious").between(age: 18..25).first
```
If you would like to use activerecord like dynamic finders then you'll have to install the gem called [Mongoid dynamic finder].
Just add the gem and bundle.

```sh
Artist.find_by_name_and_title('Some Artist' , 'Mr.')
```
So how does it work? Today I'll explain how these dyanmic finders work by explaining the mongoid dynamic finders gem.

-----------

##Where the magic happens

```sh
def method_missing(method_id, *args, &block)
      conditions = {}
      bang = false
```
Calling the find_by_name method will end up here at method_missing because there is not a method called by that name. The method_missing method initiates the conditions hash and bang variable. Method_id will be find_by_name and arguments will be 'Some Artist', 'Mr.'

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
 - Line 1 turns method name to string.
 - If method_id is acceptable by the regular expression on line 2
then the regular expression parses two groups of data from the method_id (all or last). If method_id is empty then it will be assigned as :first. 
 - If there is a bang at the end of the method call then the bang variable will be set to true, on line 4.
 - In order to find the conditions for find method the second part of parsing $2 will be split into an array. So if it is find_by_name_and_title
then the conditions hash will have keys name and title, their values will be the order of them.

```sh
        result = find(finder_type, :conditions => conditions)
```
Here is the simple part: once the finder_type and conditions information is parsed then it calls the find method on the [Mongoid] gem and saves the returned information to result.

------

**Article by [M.Cenk Ayberkin]**

[Mongoid dynamic finder]:https://github.com/mitijain123/mongoid_dynamic_finder
[Mongoid]:http://mongoid.org/en/mongoid/index.html
[M.Cenk Ayberkin]:https://github.com/cenkayberkin
