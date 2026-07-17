
## 人物移动
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class move : MonoBehaviour
{
    // Vector3 三维变量 vector:向量
    public Vector3 vector3;

    void Start()//创建时执行一次(类似与构造函数)
    { }

    void Update()//每一帧都会变化
    {
        //GetKey(...)长按
        //GetKeyDown(...)单次点按
        if(Input.GetKey(KeyCode.A)){
             transform.position += new Vector3(0.01f,0,0);
        }
        else if(Input.GetKey(KeyCode.D)){
             transform.position += new Vector3(-0.01f,0,0);
        }
        else if(Input.GetKey(KeyCode.W)){
             transform.position += new Vector3(0,0.01f,0);
        }
        else if(Input.GetKey(KeyCode.S)){
             transform.position += new Vector3(0,-0.01f,0);
        }
        
    }
}
```
## 打日志
```C#
public string log = "hello world"
void Start(){
    Debug.Log(log);
}
```
## 字段权限
* public: 公有 可以在unity编辑器中查看 其他类可以修改  
* private: 私有 仅在当前类中查看 其他类不可以修改  

如果想要用private 又允许在unity编辑器中查看 
```C#
[SerializeField] private float jumpForce;
```
## 刚体运动
Rigidbody2D:2D刚体
BoxCollider2D:2D方块碰撞器
SpriteRenderer:2D渲染器
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Move : MonoBehaviour
{
     public float speed = 10f;//移动速度
     public float jumpForce = 20f;//跳跃强度

     Rigidbody2D rigidbody2D;//刚体组件
    void Start()//创建时执行一次(类似与构造函数)
    {
         rigidbody2D = GetComponent<Rigidbody2D>();
    }

    void Update()//每一帧都会变化
    {
        MoveFun();
        JumpFun();
    }

    void MoveFun(){
     //velocity:速度
     if(Input.GetKey(KeyCode.A)){
          rigidbody2D.velocity = new Vector2(-speed,rigidbody2D.velocity.y);
     }
     else if(Input.GetKey(KeyCode.D)){
          rigidbody2D.velocity = new Vector2(speed,rigidbody2D.velocity.y);
     }
     else{
          //急停 防止漂移
          rigidbody2D.velocity = new Vector2(0,rigidbody2D.velocity.y);
     }
    }

    void JumpFun(){
     if(Input.GetKeyDown(KeyCode.W)){
          rigidbody2D.velocity = new Vector2(rigidbody2D.velocity.x,jumpForce);
     }
    }
}
```
另一种写法(适配性更强)
```C#
void MoveFun(){
     xInput = Input.GetAxisRaw("Horizontal");//unity内置的输入 获取水平输入 -1 0 1
     rigidbody2D.velocity = new Vector2(xInput * speed,rigidbody2D.velocity.y);
}
```

## animator动画
1. window -> animation -> animator
2. project -> creat -> animator controller