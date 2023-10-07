---
title: "1.贪吃蛇"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 相机设置

背景啥颜色都可以，这里大小(Size)调大点可以让蛇接下来的移动看起来更流畅，但是蛇的大小也要调整。


# 蛇的组件

在场景中创建空对象改名为Snake，添加精灵渲染器(Sprite Renderer)、刚体组件(Rigidbody 2D)和碰撞组件(Box Collider 2D)。

Spriter Renderer组件可以让图形显示出来，这里精灵(Sprite)可以简单理解为图片，设为正方形，调整颜色。

刚体组件让物体有了基本的物理特性，这里把重力(Gravity Scale)设为0，整个游戏在2D平面中所以不需要重力。

碰撞组件让物体有了碰撞体积（面积），调整碰撞区域的面积，一般都会比物体的体积小一些，不会导致多余的“误触”。

## 蛇的移动

想要让蛇移动，就需要用到脚本，在资产(Assets)中创建脚本，添加到Snake游戏对象中。
{{< expand  "Snake.cs" >}}
```c#
using UnityEngine;

public class Snake : MonoBehaviour
{
    Vector3 direction = Vector3.right;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.W) && direction != Vector3.down)
            direction = Vector3.up;
        if (Input.GetKeyDown(KeyCode.A) && direction != Vector3.right)
            direction = Vector3.left;
        if (Input.GetKeyDown(KeyCode.S) && direction != Vector3.up)
            direction = Vector3.down;
        if (Input.GetKeyDown(KeyCode.D) && direction != Vector3.left)
            direction = Vector3.right;
    }

    this.transform.position = new Vector3(
            Mathf.Round(this.transform.position.x) + direction.x,
            Mathf.Round(this.transform.position.y) + direction.y,
            0.0f
        );
}
```
{{< /expand >}}

Update()：更新事件，执行N次，每帧执行一次。

FixedUpdate()：固定更新事件，执行N次，0.02（可以设置）秒执行一次。所有物理组件相关的更新都在这个事件中处理。

在Update中移动可能会有一些问题，如果运行帧率非常高，会导致Snake的移动变得很快。所以在FixedUpdate中处理。

Mathf.Round()对其进行四舍五入，让Snake更好的与网格对齐。


> 在edit->projec setting中设置FixedUpdate的时间间隔



# 场景布置

在相机内布置四面墙壁，限制Snake的移动。添加碰撞组件设为触发器(is Trigger)，调整碰撞区域大小。
![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-06%20201010.png)


# 添加食物

和Snake一样，不过不需要刚体组件，同时设为触发器。

![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-06%20202244.png)

创建一个空对象，命名为区域(Area)添加碰撞组件，调整大小，食物在区域中随机生成，Sanke碰到食物后重新更改位置。

![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-06%20214619.png)

## 随机生成食物

创建脚本添加到食物中，接受一个碰撞对象(Area)，使其在区域(Area)内随机出现。

{{< expand  "Food.cs" >}}
```c#
public class Food : MonoBehaviour
{
    public Collider2D Area;

    private void Start()
    {
        Sprwan();
    }


    void Sprwan()
    {
        float x = Random.Range(Area.bounds.min.x, Area.bounds.max.x);
        float y = Random.Range(Area.bounds.min.y, Area.bounds.max.y);

        this.transform.position = new Vector3(Mathf.Round(x), Mathf.Round(y), 0.0f);
    }

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if(collision.tag == "Player")
        {
            Sprwan();
        }
    }
}
```
{{< /expand >}}

Start()：开始事件，执行一次。

OnTriggerEnter2D(Collider2D collision)：当另一个碰撞器进入触发器时，调用此函数。


# 蛇的生长和限制

现在Snake可以自由移动并且碰到食物会让食物随机出现。接下来就要让Snake可以生长，然后添加一些限制。

当Snake碰到墙壁或自己时重新回到出身点（屏幕中心），碰到食物则让自己生长。为了区分碰撞的物体需要设置一些Tag。

![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-06%20220657.png)
![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-06%20220649.png)

创建一个Snakebody作为预制体，可以将大小设置小点，动起来会更好看。
![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-10-06%20221953.png)


定义一个restart()函数，用来让Snake回到出生点，再定义一个Grow()用来让Snake生长。最后通过OnTriggerEnter2D()调用。

{{< expand  "Snake.cs" >}}
```c#
...
List<Transform> Snakebody = new List<Transform>();
public Transform body;
public int snake = 4;

void restart()
    {
        for(int i = 1; i<Snakebody.Count; i++)
        {
            Destroy(Snakebody[i].gameObject);
        }
        Snakebody.Clear();
        Snakebody.Add(transform);

        for(int i = 1; i<snake; i++)
        {
            Snakebody.Add(Instantiate(this.body));
        }

        transform.position = new Vector3(0.0f, 0.0f, 0.0f);

    }

void Grow()
    {
        Transform body = Instantiate(this.body);
        body.position = Snakebody[Snakebody.Count - 1].position;
        Snakebody.Add(body);
    }

private void OnTriggerEnter2D(Collider2D collision)
    {
        if(collision.tag == "Food")
        {
            Grow();
        }
        else if(collision.tag == "Wall")
        {
            restart();
        }
    }

```
{{< /expand >}}

![](https://ynsource.oss-cn-beijing.aliyuncs.com/Unity%E5%B0%8F%E6%B8%B8%E6%88%8F/Snake/1.png)
