---
layout: post
title: "Custom Scheduling in Spring"
date: 2011-11-05 19:00
comments: true
categories: Java 
---
Spring 3.0 has <a href="http://blog.springsource.com/2010/01/05/task-scheduling-simplifications-in-spring-3-0/">simplified task scheduling</a>.  As part if this, they have deprecated the MethodInvokingTimerTaskFactoryBean and ScheduledTimerTask.  Instead you create a scheduler that implements the TaskScheduler interface and uses a Trigger to specify when a task is scheduled to run.  The XML and annotations allow you to specify fixedDelay,  fixedRate or cron string.  These are fixed at run time.  This works great for triggers that are fixed at run time, but does not allow you any way to modify these at run time.  The TaskScheduler interface provides methods to schedule a task with a trigger, so this gives us an opportunity to pass in a custom trigger that can have it's trigger interval changed at run time.  There are a number of ways to configure this.  Here is a simple way I came up with that uses a single bean to schedule the task and change the fixedDelay at run time.  This extends the example provided on the Spring blog noted earlier.
<br/><br/>
First we need a class that takes the scheduler, task and starting delay.  For simplicity, it will also implement the Timer interface.
<pre><span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">DynamicSchedule</span> <span style="color:#a020f0;">implements</span> <span style="color:#228b22;">Trigger</span> {

<span style="color:#a020f0;">   private</span> <span style="color:#228b22;">TaskScheduler</span> scheduler;
   <span style="color:#a020f0;">private</span> <span style="color:#228b22;">Runnable</span> task;
   <span style="color:#a020f0;">private</span> <span style="color:#228b22;">ScheduledFuture</span>&lt;?&gt; future;
   <span style="color:#a020f0;">private</span> <span style="color:#a020f0;">int</span> delay;

   <span style="color:#a020f0;">public</span> <span style="color:#228b22;">DynamicSchedule</span>(<span style="color:#228b22;">TaskScheduler</span> scheduler, <span style="color:#228b22;">Runnable</span> task, <span style="color:#a020f0;">int</span> delay) {
      <span style="color:#a020f0;">this</span>.scheduler = scheduler;
      <span style="color:#a020f0;">this</span>.task = task;
      reset(delay);
   }

   <span style="color:#a020f0;">public</span> <span style="color:#a020f0;">void</span> reset(<span style="color:#a020f0;">int</span> delay) {
      <span style="color:#a020f0;">if</span> (future != null) {
         <span style="color:#228b22;">System</span>.out.println(<span style="color:#bc8f8f;">"Cancelling task..."</span>);
         future.cancel(<span style="color:#b8860b;">true</span>);
      }
      <span style="color:#a020f0;">this</span>.delay = delay;
      <span style="color:#228b22;">System</span>.out.println(<span style="color:#bc8f8f;">"Starting task..."</span>);
      future = scheduler.schedule(task, <span style="color:#a020f0;">this</span>);
   }

<span style="color:#b8860b;">   @Override</span>
   <span style="color:#a020f0;">public</span> <span style="color:#228b22;">Date</span> nextExecutionTime(<span style="color:#228b22;">TriggerContext</span> triggerContext) {
      <span style="color:#228b22;">Date</span> lastTime = triggerContext.lastActualExecutionTime();
      <span style="color:#228b22;">Date</span> nextExecutionTime = (lastTime == null)
         ? <span style="color:#a020f0;">new</span> <span style="color:#228b22;">Date</span>()
         : <span style="color:#a020f0;">new</span> <span style="color:#228b22;">Date</span>(lastTime.getTime() + delay);
         System.out.println("DynamicSchedule -- delay: " + delay +
              ", lastActualExecutionTime: " + lastTime +
              "; nextExecutionTime: " + nextExecutionTime);
      return nextExecutionTime;
   }

}</pre><p>Note the reset method which stops the scheduled task, changes the delay and then restarts the task.  If you are changing the delay to a shorter delay, you want to restart with the new delay so it happens immediately.  Alternately, you can skip canceling the task and the new delay is picked up on the next execution.
<br/><br/>
The rest of the code is the same, except for the SchedulerProcessor which has the @Scheduled annotation removed from the process method:<p><pre><span style="color:#b8860b;">@Component</span><span style="color:#a020f0;">
public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">ScheduledProcessor</span> {

<span style="color:#a020f0;">   private</span> <span style="color:#a020f0;">final</span> <span style="color:#228b22;">AtomicInteger</span> counter = <span style="color:#a020f0;">new</span> <span style="color:#228b22;">AtomicInteger</span>();

<span style="color:#b8860b;">   @Autowired
   </span><span style="color:#a020f0;">private</span> <span style="color:#228b22;">Worker</span> worker;

<span style="color:#a020f0;">   public</span> <span style="color:#a020f0;">void</span> process() {<span style="color:#228b22;">
      System</span>.out.println(<span style="color:#bc8f8f;">"processing next 10 at "</span> + <span style="color:#a020f0;">new</span> <span style="color:#228b22;">Date</span>());
      <span style="color:#a020f0;">for</span> (<span style="color:#a020f0;">int</span> i = 0; i &lt; 10; i++) {
         worker.work(counter.incrementAndGet());
      }
   }

}</pre><p>In the XML configuration, we add a name to the scheduler and create the DynamicSchedule. We pass it the scheduler, the process method (wrapped in a MethodInvokingRunnable) and the default delay:</p><pre>   &lt;<span style="color:#000000;">context</span>:<span style="color:#0000ff;">component-scan</span> <span style="color:#b8860b;">base-package</span>=<span style="color:#bc8f8f;">"com/test"</span> /&gt;

   &lt;<span style="color:#000000;">task</span>:<span style="color:#0000ff;">annotation-driven</span> /&gt;

   &lt;<span style="color:#000000;">task</span>:<span style="color:#0000ff;">scheduler</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"scheduler"</span> /&gt;

   &lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">id</span>=<span style="color:#bc8f8f;">"dynamicSchedule"</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"com.test.DynamicSchedule"</span>&gt;
      &lt;<span style="color:#0000ff;">constructor-arg</span> <span style="color:#b8860b;">ref</span>=<span style="color:#bc8f8f;">"scheduler"</span> /&gt;
      &lt;<span style="color:#0000ff;">constructor-arg</span>&gt;
         &lt;<span style="color:#0000ff;">bean</span> <span style="color:#b8860b;">class</span>=<span style="color:#bc8f8f;">"org.springframework.scheduling.support.MethodInvokingRunnable"</span>&gt;
            &lt;<span style="color:#0000ff;">property</span> <span style="color:#b8860b;">name</span>=<span style="color:#bc8f8f;">"targetObject"</span> <span style="color:#b8860b;">ref</span>=<span style="color:#bc8f8f;">"scheduledProcessor"</span> /&gt;
            &lt;<span style="color:#0000ff;">property</span> <span style="color:#b8860b;">name</span>=<span style="color:#bc8f8f;">"targetMethod"</span> <span style="color:#b8860b;">value</span>=<span style="color:#bc8f8f;">"process"</span> /&gt;
         &lt;/<span style="color:#0000ff;">bean</span>&gt;
      &lt;/<span style="color:#0000ff;">constructor-arg</span>&gt;
      &lt;<span style="color:#0000ff;">constructor-arg</span> <span style="color:#b8860b;">value</span>=<span style="color:#bc8f8f;">"3000"</span> /&gt;
   &lt;/<span style="color:#0000ff;">bean</span>&gt;</pre><p>Now we can add a separate process that changes the delay to a random delay to test it out:</p><pre><span style="color:#b8860b;">@Component</span>
<span style="color:#a020f0;">public</span> <span style="color:#a020f0;">class</span> <span style="color:#228b22;">ScheduleChanger</span> {

<span style="color:#b8860b;">   @Autowired</span>
<span style="color:#a020f0;">   private</span> <span style="color:#228b22;">DynamicSchedule</span> dynamicSchedule;

<span style="color:#b8860b;">   @Scheduled</span>(fixedDelay=30000)
<span style="color:#a020f0;">   public</span> <span style="color:#a020f0;">void</span> change() {
<span style="color:#228b22;">      Random</span> rnd = <span style="color:#a020f0;">new</span> <span style="color:#228b22;">Random</span>();
      <span style="color:#a020f0;">int</span> nextTimeout = rnd.nextInt(30000);
<span style="color:#228b22;">      System</span>.out.println(<span style="color:#bc8f8f;">"Changing poll time to: "</span> + nextTimeout);
      dynamicSchedule.reset(nextTimeout);
   }

}</pre><p>When you run this and view the output, you will see where the dynamic schedule trigger is fired and where the schedule gets changed.</p>
