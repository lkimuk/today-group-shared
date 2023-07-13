## T230705 Complex C++
提到 C++，许多人的印象绝对不会少了一个词：复杂。

是啊！2000 页左右的标准，甚至一些特性单独就能成一本书，怎么能不复杂？

但也别忘了它是一门接近 40 年的编程语言。

系统大了，问题本身就很复杂，很多时候并不存在完美的解决方案。很多问题的一面是另一个系统的另一面，解决这个问题必然会带来另一个问题。所以许多时候都要分清主次，把主要模板的问题潜移默化地排到其他次要模块中去，解决当下的问题。

问题一直都没有消失，只是在这个大系统中滚来滚去。要想真正解决问题，就得不断寻求增量，增加新特性，很多旧的问题就这样通过增量解决了。随之而来的又是新的问题，循环往复~

新特性使用起来更方便，很多时候就是因为从更高维度解决了问题。一些新语言也只是暂时把一些问题转移到了次要方面，时间一长照样暴露无遗。也正是因为总是追求完美，而问题本身又极具复杂性，且缺少许多前置组件，又要兼容旧的，所以标准才加得超慢。

不同定制实现就是在矛盾中自己决定主次，想一招吃遍天下不可能。默认定制适用范围越广，特性用起来就越简单。但一旦换个场景，增加定制是否也这么简单，就不好说了。

Bjarne 在 Herb 的某次演讲最后说：
> there's a huge chunk of the C++ community that love complexity. They want every little detail of the implementation being obvious, so that they know exactly how many nanoseconds it might cost. Sometimes I'm on that team, but 99% of the time I am not and I'm suffering from the most programmers who can't figure out what things means, who can't read the manual and make all of those bugs... but notice we got complexity over the standards committee, we got these redundant autos in concepts, we've got three different interfaces to something that is fundamentally very similar, optional, variant, and any. we got complications to dynamic binding which was explicitly part of the path to pattern matching and of course we have an endless number of casts.... as simple as uniform and direct as possible and leave the complications to code that we already write today.

问题本身是复杂的，但我们不要沉迷于复杂本身，而是想着怎样将复杂藏于身后，提供唯一、直接、简单、安全的东西。

复杂不是目的，简单才是。