---
layout: post
category : lessons
title: "meteor发布与订阅"
tagline: "Supporting tagline"
tags : [meteor]
---

Meteor服务端可以通过`Meteor.publish`发布一组documents，客户端可以通过`Meteor.subscribe` 订阅这些文档。客户端订阅的所有的documents都可以通过客户端的collections的`find`方法查询到。

默认情况下，所有新创建的Meteor的app都包含autopublish包，它会自动发布所有documents到每一个客户端。 要想更细粒度的控制每个客户端收到的documents,首先移除autopublish

```
$ meteor remove autopublish
```

现在你就可以通过`Meteor.publish`和`Meteor.subscribe`来控制服务端发送哪些document 到客户端。

```
Meteor.publish(name, func) 				定义在Server
	发布一个记录集.

	Arguments
		name String
		记录集的名字。如果为 null, 则记录集没有名字，并且该记录集会自动发布到所有已连接上的客户端。

		func Function
		每次有客户端订阅时，调用该函数。在函数中， this 代表publish handler对象, 
		下面有介绍。如果客户端传递了参数给subscribe, 该函数会以相同参数被调用。
```

要发布数据到客户端，在服务端调用`Meteor.publish`，带上两个参数：记录集的名字，和一个 publish function，每次有客户端订阅这个记录集的时候，这个函数就会被调用。

publish函数通常对某些collection执行query，返回调用`collection.find(query)`的结果：

```
// Publish the logged in user's posts
Meteor.publish("posts", function () {
  return Posts.find({ createdBy: this.userId });
});
```

可以通过返回一个collection.find数组来发布多个collections到documents：

```
// Publish a single post and its comments
Meteor.publish("postAndComments", function (postId) {
  // Check argument
  check(postId, String);

  return [
    Posts.find({ _id: postId }),
    Comments.find({ postId: roomId })
  ];
});
```

在publish函数中，`this.userId`代表当前登录用户的_id，可以用来过滤collections,使 特定的documents只对特定的用户可见。如果登录用户切换到另一个账号，publish函数会使用新的 userId自动重运行，所以当前用户获取不到之前登录用户的documents。

```
Meteor.subscribe(name, [arg1, arg2...], [callbacks]) 	定义在Client
	订阅一个记录集，返回一个handler，handler提供stop()和ready()方法。

	Arguments
		name String
		订阅的名字，要和服务端的publish()的名字一致

		arg1, arg2... Any
		可选的参数，会传递给服务端的publish函数。

		callbacks Function or Object
		可选。可以包含onStop和onReady回调函数。如果发生错误，会作为参数传递给 onStop. 如果callbacks是个函数形式而不是对象, 函数会被解释为onReady 。
```

客户端调用`Meteor.subscribe`来获取服务端发布的documents。客户端可以通过调用collection.find(query) 来进一步过滤documents。如果publish函数发布的数据发生变化时，publish函数会自动重新运行然后 更新已经推送到客户端的document

当服务端把所有初始数据发送给订阅后，onReady回调函数就会被调用，不带参数。当订阅因为任何原因 被终止后onStop回调函数就会被调用，如果订阅是由于服务端错误导致的失败的话，它会收到一个`Meteor.Error`

Meteor.subscribe返回一个订阅handle,handle 对象包含下面两个方法：

```
	stop()
	取消订阅。通常结果是服务端指示客户端从客户端缓存中移除订阅数据。

	ready()
	如果服务端标记订阅为ready，则返回true。一个reactive数据源。
```
If you call Meteor.subscribe inside Tracker.autorun, the subscription will be cancelled automatically whenever the computation reruns (so that a new subscription can be created, if appropriate), meaning you don't have to to call stop on subscriptions made from inside Tracker.autorun.
















