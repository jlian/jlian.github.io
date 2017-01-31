---
layout: post
title: "Efficient Animation With MATLAB"
excerpt: "A guide to fast, flexible animation techniques in MATLAB"
tags: [MATLAB, thesis]
<!-- image:
  feature: latech.jpg -->
---

I used animation to help me visualize some of the work I did for my [honours thesis](/projects/thesis.pdf) (PDF warning). Prior to [MATLAB R2014b's major graphics changes](http://www.mathworks.com/help/matlab/graphics-changes-in-r2014b.html), it wasn't exactly easy to create efficient animations with MATLAB, so I spent some time figuring out the best way to do it. This is what I had in the end:

<figure>
	<a href="/matlab-animation/shock.gif"><img src="/matlab-animation/shock.gif"></a>
	<figcaption>Shockwave created by a driving piston in a one-dimensional nonlinear lattice</figcaption>
</figure>

### The "Easy" Way

Say you want to animate a particle trajectory (position vs time), the most obvious way would be to use `plot()` in a `for` loop, kind of like this:

{% highlight matlab %}
% Animation loop
for i = 1:totalFrames
	% Compute the necessary data
	particlePosition(i) = rand;

	% Plot the result of the computation
	plot(timeFrame(i), particlePosition(i));
end
{% endhighlight %}

If you've ever tried doing this you would notice that it does not scale well. Even if the plot is just a little bit complex this method becomes *very slow*, with lots of flashes of the figure window. Every time `plot()` is called MATLAB has to repeat a lot of unnecessary work.[^1] 

[^1]: This is no longer the case after R2014b.

### A Better Method

In [MathWorks' article on animation techniques](http://www.mathworks.com/help/matlab/creating_plots/animation-techniques.html), we can see that they recommend 

> Update the properties of a graphics object and display the updates on the screen. This technique is useful for creating animations when most of the graph remains the same. For example, set the `XData` and `YData` properties repeatedly to move an object in the graph.

Instead of using `plot()`, the `set()` function skips a lot of unnecessary work, making it more efficient. An implementation example would look like this:

{% highlight matlab %}
% Initialize the plot
h = plot(timeFrame(1), particlePosition(1));

% Animation loop
for i = 2:totalFrames
	% Compute the necessary data
	particlePosition(i) = rand;

	% Change the data in the plot
	set(h, 'XData', timeFrame(i));
	set(h, 'YData', particlePosition(i));
end
{% endhighlight %}

**Note:** Sometimes, especially if your animation update command is after complicated computation, you need to use `drawnow` to force the animation to occur in real time.
{: .notice}

#### A Quick Example

I wrote a simple script that uses this technique to animate a particle in a sine trajectory. It should look like this:[^2]

<figure>
	<a href="/matlab-animation/sine.gif"><img src="/matlab-animation/sine.gif"></a>
	<figcaption>A particle with sine trajectory</figcaption>
</figure>

<a markdown="0" href="{{ site.url }}/matlab-animation/Animate.m" class="btn">Download Animate.m</a>

[^2]: This GIF is 60 FPS.

### Animating Multiple Trajectories

I ran into some problems when I needed to animate *variable* number of trajectories. I wanted to visualize how changing the number of particles in my system changed the trajectories, but MATLAB was giving me errors when I used the `set()` method. 

It turns out that animating multiple trajectories in stored one variable[^3] with the *same time vector* requires a trick. The old method won't work if the number of trajectories will be changing all the time. I will illustrate with an example:

[^3]: Because of the way `ode45` worked, it was most convenient this way.

{% highlight matlab %}
% Initialize the trajectories
particlePosition = zeros(randi(5),totalFrames)

% Initialize the plot
h = plot(timeFrame(1), particlePosition(:,1));

% Animation loop
for i = 2:totalFrames
	% Compute the necessary data
	for j = 1:size(particlePosition,1)
		particlePosition(j,i) = rand;
	end

	% Change the data in the plot
	set(h, 'XData', timeFrame(i));
	set(h, {'YData'}, num2cell(particlePosition(:,i)));
end
{% endhighlight %}

Here, `particlePosition` contains up to five trajectories. Note the use of `num2cell` in setting `YData` is required for the animation to work properly because of the way graphics data are structured in MATLAB. 

**Note:** The`YData` you are setting must be a **column cell vector**. So if your data is structured such that each row represents a frame in the animation, you must transpose your data in `set()`.
{: .notice}


#### Another Quick Example

You can download the other script to see how this can be implemented. It looks like this:

<figure>
	<a href="/matlab-animation/sine2.gif"><img src="/matlab-animation/sine2.gif"></a>
	<figcaption>Three particles with different trajectories</figcaption>
</figure>

<a markdown="0" href="{{ site.url }}/matlab-animation/Animate2.m" class="btn">Download Animate2.m</a>

### An Example from MathWorks

I learned a lot from MathWorks' animation example here: <https://www.mathworks.com/examples/matlab/4020-animation>, including how to export as GIFs.


