# design our game engine
游戏引擎一般需要这些：  
* Entrypoint
* Application layer
* Window layer
  * Input
  * Event
* Renderer
* Render API abstraction:最开始用OpenGL，简单，如果要支持多种库，需要好好设计API
* Debugging support：logging system
* Script language
* Entity component system
* file io,vfs
# project setup
github rep create  
vs local project create  
我们的目标是将引擎做成一个动态库，可以链接到目标中。不使用静态链接是因为这个时候每个使用这个引擎的游戏都需要把引擎所依赖的库链接到游戏里。而动态链接不需要游戏文件去做这个事情，引擎自己把依赖链接起来就行了。  
在solution中再新建一个Sandbox项目 
<!--more-->
