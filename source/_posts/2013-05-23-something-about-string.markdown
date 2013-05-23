---
layout: post
title: "关于String的一个问题"
date: 2013-05-23 21:54
comments: true
categories: 
---
*	先看一段源代码：
		 /** The value is used for character storage. */
		private final char value[];

		/** The offset is the first index of the storage that is used. */
		private final int offset;

		/** The count is the number of characters in the String. */
		private final int count;

	
		/**
		* Initializes a newly created {@code String} object so that it represents
		* the same sequence of characters as the argument; in other words, the
		* newly created string is a copy of the argument string. Unless an
		* explicit copy of {@code original} is needed, use of this constructor is
		* unnecessary since Strings are immutable.
		*
		* @param  original
		* 	A {@code String}
		*/
		public String(String original) {
			int size = original.count;
			char[] originalValue = original.value;
			char[] v;
			if (originalValue.length > size) {
				// The array representing the String is bigger than the new
				// String itself.  Perhaps this constructor is being called
				// in order to trim the baggage, so make a copy of the array.
				int off = original.offset;
				v = Arrays.copyOfRange(originalValue, off, off+size);
			} else {
				// The array representing the String is the same
				// size as the String, so no point in making a copy.
				v = originalValue;
			}
			this.offset = 0;
			this.count = size;
			this.value = v;
		}
	&nbsp;&nbsp;&nbsp;&nbsp;这是 java.lang.String 的一部分源代码。其中有一个构造函数 public String(String original)  用一个  String去构造另一个函数，大部分时间这个构造函数是没有用的，正如注解里写的那样Unless an explicit copy of {@code original} is needed, use of this constructor isunnecessary since Strings are immutable.
	
	&nbsp;&nbsp;&nbsp;&nbsp;不过，我们暂时不必关系这个构造函数存在的意义。先看看里面的代码 originalValue.length>size 字符串持有的字符数组的长度，比这个字符串的长度要大。再看看一开始对count的定义，count 就是这个String对象里字符的个数/** The count is the number of characters in the String. */,既然如此，那么为什么count大小又和
	字符数组的长度不一样了呢?
	
	&nbsp;&nbsp;&nbsp;&nbsp;于是，开始google，搜了老半天，终于有了一点眉目。说是，这个字符数组是有可能被很多的String共享的，在共享同一个 char[] value的时候，就有可能因为每个对象实际拥有的字符数量不同而修改count了。
	听着好像那么一回事，但是还觉得有些地方总是困惑着自己。那就是，什么情况下，可能会共享这个value呢？能不能找出来一个例子呢？于是，开始各种构造字符串，尝试了几种方法之后，还是没有结果。徒劳。
	
	&nbsp;&nbsp;&nbsp;&nbsp;再静下心来，仔细想想，既然是共享一个value，count是指字符个数，那么就是通过count，offset来区分每个String对象。那么offset一定不是等于0!!于是在源码里面搜索"offset =", 终于被我搜到下面的三段代码：
		public String(StringBuffer buffer) {
			String result = buffer.toString();
			this.value = result.value;
			this.count = result.count;
			this.offset = result.offset;
		}
		public String(StringBuilder builder) {
			String result = builder.toString();
			this.value = result.value;
			this.count = result.count;
			this.offset = result.offset;
		}
		// Package private constructor which shares value array for speed.
		String(int offset, int count, char value[]) {
			this.value = value;
			this.offset = offset;
			this.count = count;
		}
	&nbsp;&nbsp;&nbsp;&nbsp;看到第三个 私有构造方法上的注解了吗!!// Package private constructor which shares value array for speed. 终于找到了!看看是怎么具体实现的，查看调用者，有一个subString(int beginIndex,int endIndex),看看这个方法的实现：
		 public String substring(int beginIndex, int endIndex) {
			if (beginIndex < 0) {
				throw new StringIndexOutOfBoundsException(beginIndex);
			}
			if (endIndex > count) {
				throw new StringIndexOutOfBoundsException(endIndex);
			}
			if (beginIndex > endIndex) {
				throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
			}
			return ((beginIndex == 0) && (endIndex == count)) ? this :
				new String(offset + beginIndex, endIndex - beginIndex, value);
			}
		}
	&nbsp;&nbsp;&nbsp;&nbsp;当你的构造一个subString 不是它本身的时候，他就new String(offset + beginIndex, endIndex - beginIndex, value);这里的offset 就不是0了，count也不是value的length，但是value还是共享了父String的value，这样就出现了共享了字符数组，并且
	数组长度和count不一致。
	
	&nbsp;&nbsp;&nbsp;&nbsp;写一个小例子测试，并且单步调试一下（因为value和count都是私有变量，不这么做看不到）：
		public static void main(String[] args) {
			String b = "abcdefghit";
			String sub = b.substring(2, 5);
			String tt = new String(sub);
			System.out.println(tt);
		}
	&nbsp;&nbsp;&nbsp;&nbsp;结果如下图：
	![Alt text](https://raw.github.com/chunlong/chunlong.github.com/master/images/result.png)
	&nbsp;&nbsp;&nbsp;&nbsp;看到b.value的id和sub.value的id是一致了。说明确实是共享一个char 数组。并且，sub的count也是3而不是10.至此都明白了。
	