# Scala

## 1. 半生类和半生对象

```scala
//半生类
class A{
def apply()={

}

}

//半生对象
object A{

def apply()={

}

}


val a=A()   //调用的是object.apply
val a1=new A()
al()       //调用的是class.apply


//类名()    object.apply
//对象名()  class.apply


最佳实践是在object的apply里面 new Class
```



## 2. 尾递归求和

```scala
def sum(nums:Int*)={
	if(nums==0){
		0
	}else{
		nums.head+sum(nums.tail:_*)
	}

}
```



## 3. Range

to // 闭区间

until //左闭右开

Range //左闭右开

```
1 to 4 			 => [1,4]  =>  1,2,3,4
1 until 4 	 => [1,4)  =>  1,2,3
Range(1,4)   => [1,4)  =>  1,2,3
```



