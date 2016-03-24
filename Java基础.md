## Java Collection
* Collection 比数组好 :将数组转换为Collection

		String[] args = {"nimiw","fuck"}；
		List<String> argList = Arrays.asList(args);
* 迭代器的低效率
	* 尽量不要对集合进行迭代，缺点如下：
		* 每次添加或移除元素后重新调整集合将非常低效
		* 每次在获取锁、执行操作和释放锁的过程中，都存在潜在的并发困境
		* 当添加或移除元素时，存取集合的其他线程会引起竞争条件
	* 通过使用 `addAll` 或 `removeAll`，传入包含要对其添加或移除元素的集合作为参数，来避免所有这些问题
* 用 for 循环遍历任何 Iterable

		//Person.java
		import java.util.*;
		public class Person implements Iterable<Person>
		{	
    		public Person(String fn, String ln, int a, Person... kids)
    		{
    	  	 	this.firstName = fn; this.lastName = ln; this.age = a;
    	   		for (Person child : kids)
    	        children.add(child);
    		}
	    	public String getFirstName() { return this.firstName; }
	    	public String getLastName() { return this.lastName; }
	    	public int getAge() { return this.age; }
	    	
	    	public Iterator<Person> iterator() { return children.iterator(); }
	    	
	    	public void setFirstName(String value) { this.firstName = value; }
	    	public void setLastName(String value) { this.lastName = value; }
	    	public void setAge(int value) { this.age = value; }
	    	
	    	public String toString() { 
	    	    return "[Person: " +
	    	        "firstName=" + firstName + " " +
	    	        "lastName=" + lastName + " " +
	    	        "age=" + age + "]";
	    		}
    
    		private String firstName;
    		private String lastName;
    		private int age;
    		private List<Person> children = new ArrayList<Person>();
		}

		// App.java
		public class App{
	    	public static void main(String[] args)
	    	{
	        	Person ted = new Person("Ted", "Neward", 39,
	            new Person("Michael", "Neward", 16），
	            new Person("Matthew", "Neward", 10));
	
	        	// Iterate over the kids
	        	for (Person kid : ted)
	        	{
	            	System.out.println(kid.getFirstName());
	        	}
	    	}
		}
* 扩展 Collections API

		public interface SortedCollection<E> extends Collection<E>
		{
    		public Comparator<E> getComparator();
    		public void setComparator(Comparator<E> comp);
		}

		public class ArraySortedCollection<E> 
			implements SortedCollection<E>, Iterable<E>
		{
		    private Comparator<E> comparator;
		    private ArrayList<E> list;
		        
		    public ArraySortedCollection(Comparator<E> c)
		    {
		        this.list = new ArrayList<E>();
		        this.comparator = c;
		    }
		    public ArraySortedCollection(Collection<? extends E> src, Comparator<E> c)
		    {
		        this.list = new ArrayList<E>(src);
		        this.comparator = c;
		        sortThis();
		    }
		
		    public Comparator<E> getComparator() { return comparator; }
		    public void setComparator(Comparator<E> cmp) { comparator = cmp; sortThis(); }
		    
		    public boolean add(E e)
		    { boolean r = list.add(e); sortThis(); return r; }
		    public boolean addAll(Collection<? extends E> ec) 
		    { boolean r = list.addAll(ec); sortThis(); return r; }
		    public boolean remove(Object o)
		    { boolean r = list.remove(o); sortThis(); return r; }
		    public boolean removeAll(Collection<?> c)
		    { boolean r = list.removeAll(c); sortThis(); return r; }
		    public boolean retainAll(Collection<?> ec)
		    { boolean r = list.retainAll(ec); sortThis(); return r; }
		    
		    public void clear() { list.clear(); }
		    public boolean contains(Object o) { return list.contains(o); }
		    public boolean containsAll(Collection <?> c) { 
				return list.containsAll(c); 
			}
		    public boolean isEmpty() { return list.isEmpty(); }
			.......

		    public boolean equals(Object o)
		    {
		        if (o == this)
		            return true;
		        if (o instanceof ArraySortedCollection)
		        {
		            ArraySortedCollection<E> rhs = (ArraySortedCollection<E>)o;
		            return this.list.equals(rhs.list);
		        }
		        return false;
		    }
		    public int hashCode()
		    {
		        return list.hashCode();
		    }
		    public String toString()
		    {
		        return list.toString();
		    }
		    
		    private void sortThis()
		    {
		        Collections.sort(list, comparator);
		    }
		}

* 永远不要将可变对象类型用作 `HashMap` 中的键