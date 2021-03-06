[image1]: https://user-images.githubusercontent.com/23042512/48657451-cc597f00-e9e5-11e8-8332-bf97ee7da5f8.gif "Trained Agent Perf"
[image2]: https://user-images.githubusercontent.com/23042512/48657452-cc597f00-e9e5-11e8-8776-37a144f24702.png "Trained Agent Scores"

## Introduction
![Trained Agent][image1]


This report outlines my implementation for Udacity's Deep Reinforcement Learning Nanodegree's second project on the Reacher environment. In this project, the goal is to train an acrobat arm that has two joints so that it tracks a balloon. As the balloon moves, the two joints at adjusted to track the balloon. So this is a classical robotics problem, and using model-free reinforcement learning, the agent will learn the optimal policy. In particular, the method used is the deep deterministic policy gradient method (DDPG).

Value based reinforcement learning algorithms such as DQN have shown great performance in many domains. However, they are still limited to discrete action space environments and for deterministic policies (as they are essentially based upon a deterministic greedy policy as epsilon only selects a uniform-random action). Moreover, with value based methods, we first compute the value-function for each state, and use that to determine the best policy. This is an indirect way of finding the optimal policy.

Using a policy based method, on the other hand, we directly find the policy that yields the most rewards. Policy gradient is one of the more efficient policy based learning algorithms where we directly compute the gradient of the expected reward with respect to the policy parameters. In addition to being direct, it works well with continuous actions as well as stochastic policies. Other policy based methods include stochastic optimization methods such as random shooting, cross entropy method, etc.

Before delving into my implementation for this project, I have included below some basics on policy gradient methods. In particular, this article will walk the reader from the basic objective of Reinforcement Learning (RL) to some of the advanced policy-gradient algorithms, such as Reinforce, Actor-Critic, Advantage Actor-Critic, Deterministic Policy Gradient, and Deep Deterministic Policy Gradient. For thorough understanding, it is assumed the reader is well versed in Probability & Statistics, Linear Algebra, Vector Calculus, and basic Reinforcement Learning terminologies.

---------------------------------------------------------------
## Reinforce Algorithm
Please note, in the below analyses, the discount factor &gamma; is assumed to be 1 for simplicity. But all the analyses can be easily extended to cases where &gamma; is not 1.

The basic objective in all of Reinforcement Learning (RL) is to maximize the expected total utility U<sub>&theta;</sub>, which is defined as follows [1]:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;U_{\theta}&space;=&space;E_{\tau&space;\,&space;\sim&space;\,&space;P_{\theta}(\tau)}&space;\left&space;[&space;\sum_{t=0}^{T}r(s_{t},a_{t})&space;\right&space;],&space;whereby&space;\,&space;\tau&space;=&space;\left&space;\{&space;s_{0},&space;a_{0},&space;s_{1},&space;a_{1},&space;...,&space;s_{T},&space;a_{T}&space;\right&space;\}&space;\:&space;...&space;\,&space;Equation&space;\,&space;1a" title="U_{\theta} = E_{\tau \, \sim \, P_{\theta}(\tau)} \left [ \sum_{t=0}^{T}r(s_{t},a_{t}) \right ], whereby \, \tau = \left \{ s_{0}, a_{0}, s_{1}, a_{1}, ..., s_{T}, a_{T} \right \} \: ... \, Equation \, 1a" />

After doing some Math, it can be shown that U<sub>&theta;</sub> is equal to the expected value of Q(s<sub>0</sub>,a<sub>0</sub>). And if the initial state distribution is uniform, then it means the goal in RL is to find a policy which maximizes the q-values of all possible states.

<img src="https://latex.codecogs.com/png.latex?\dpi{120}&space;\fn_cm&space;U_{\theta}&space;=&space;E_{s_{0}&space;\,&space;\sim&space;\,&space;P(s_{0})}&space;\left&space;[&space;E_{a_0&space;\,&space;\sim&space;\,&space;P_{\theta}(a_0&space;|&space;s_0)}&space;\left&space;[&space;Q_{P_{\theta}}(s_{0},&space;a_{0})&space;|s_0&space;\right&space;]&space;\right&space;]&space;\:&space;...&space;\,&space;Equation&space;\,&space;1b" title="U_{\theta} = E_{s_{0} \, \sim \, P(s_{0})} \left [ E_{a_0 \, \sim \, P_{\theta}(a_0 | s_0)} \left [ Q_{P_{\theta}}(s_{0}, a_{0}) |s_0 \right ] \right ] \: ... \, Equation \, 1b" />

Using the definition of expectation, the above equation 1a can be re-written as:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;U_{\theta}&space;=&space;\sum_{\tau}&space;\left&space;(P_{\theta}(\tau)&space;\left(\sum_{t=0}^{T}r(s_{t},a_{t})\right)&space;\right&space;)&space;...&space;\:&space;Equation&space;\:&space;2" title="U_{\theta} = \sum_{\tau} \left (P_{\theta}(\tau) \left(\sum_{t=0}^{T}r(s_{t},a_{t})\right) \right ) ... \: Equation \: 2" />

Using Policy gradient method, we can maximize U<sub>&theta;</sub> by first computing its gradient with respect to &theta;, which can readily be derived to be:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;=&space;E_{\tau&space;\,&space;\sim&space;\,&space;P_{\theta}(\tau)}&space;\left[&space;\sum_{t=0}^{T}\left(\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\left(&space;\sum_{t=0}^{T}r(s_{t},a_{t})&space;\right)&space;\right)&space;\right]&space;\:&space;...&space;\,&space;Equation&space;\,&space;3" title="\nabla_{\theta}U_{\theta} = E_{\tau \, \sim \, P_{\theta}(\tau)} \left[ \sum_{t=0}^{T}\left(\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \left( \sum_{t=0}^{T}r(s_{t},a_{t}) \right) \right) \right] \: ... \, Equation \, 3" />

One approach to improving the expected total reward is to randomly add noise to the current &theta; and if it results in better total reward, then we keep it, otherwise we ignore it, and we keep repeating this process. This method is called the random shooting method. There are other more sophisticated methods in the same vein such as the Cross Entropy Method. All these methods fall under the domain of stochastic optimization algorithms. However, while these methods are very simple to implement, they are not efficient and don't scale well with high dimensional space. A more efficient approach is to change &theta; in the direction of the gradient using Stochastic Gradient Ascent as follows:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\theta&space;\leftarrow&space;\theta&space;&plus;&space;\alpha\nabla_{\theta}U_{\theta}&space;\;&space;...&space;\;&space;Equation&space;\,&space;4" title="\theta \leftarrow \theta + \alpha\nabla_{\theta}U_{\theta} \; ... \; Equation \, 4" />

A basic policy gradient algorithm making use of the above gradient is known as the Reinforce algorithm, and here is how it works:

***A Basic Reinforce Algorithm:***

Start with a random vector &theta; and repeat the following 3 steps until convergence:

  1. Use the policy P<sub>&theta;</sub>(a<sub>t</sub>|s<sub>t</sub>) to collect m trajectories {&tau;<sup>1</sup>, &tau;<sup>2</sup>, ..., &tau;<sup>m</sup>}, where each trajectory is as defined above.
  2. Use these trajectories to compute the Monte-Carlo estimator of the gradient as follows:
      
      <img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;\approx&space;\hat{g}&space;=&space;\frac{1}{m}&space;\sum_{i=1}^{m}&space;\left(&space;\sum_{t=0}^{T}&space;\left(&space;\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\left(&space;\sum_{t=0}^{T}r(s_{t},a_{t})&space;\right)&space;\right)&space;\right)_i&space;\:&space;...&space;\,&space;Equation&space;\,&space;5" title="\nabla_{\theta}U_{\theta} \approx \hat{g} = \frac{1}{m} \sum_{i=1}^{m} \left( \sum_{t=0}^{T} \left( \nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \left( \sum_{t=0}^{T}r(s_{t},a_{t}) \right) \right) \right)_i \: ... \, Equation \, 5" />

      Note that the reason why the above estimator is valid is because the trajectories are generated by following the policy being learned, i.e. P<sub>&theta;</sub>(&tau;) -- i.e. it is an on-policy algorithm. Another way to say it is that we sample each of the trajectories in {&tau;<sup>1</sup>, &tau;<sup>2</sup>, ..., &tau;<sup>m</sup>} from the probability distribution P<sub>&theta;</sub>(&tau;).

  3. Update the weights/parameters of the policy network using the above estimator of the gradient:

      <img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\theta&space;\leftarrow&space;\theta&space;&plus;&space;\alpha\hat{g}&space;\;&space;...&space;\;&space;Equation&space;\,&space;6" title="\theta \leftarrow \theta + \alpha\hat{g} \; ... \; Equation \, 6" />

The intuition behind the reinforce algorithm is that if the total reward is positive, then all the actions taken in that trajectory are reinforced whereas if the total reward is negative, then all the actions taken in the trajectory are inhibited. Moreover, to be computationally efficient, typically m is set to 1.

While better than stochastic optimization methods, the Reinforce algorithm suffers from a few drawbacks:
  1. The gradient estimator is pretty noisy, especially for the case m=1, because a single trajectory maynot be representative of the policy.
  2. There is no clear credit assignment. A trajectory may contain many good and bad actions, and whether those actions are reinforced or not depend only on the total reward achieved starting from the initial state.
  3. It is very sensitive to the absolute value of the rewards. For example, adding a fixed constant to all the rewards can drastically change the behavior of the algorithm. Such a trivial transformation should have no effect on the optimal policy.

By the definition of the gradient, &nabla;<sub>&theta;</sub>U<sub>&theta;</sub> points in the direction of the maximum change in U<sub>&theta;</sub>. However, at a fundamental level, the above drawbacks of Reinforce algorithm are due to the fact that the Monte-Carlo estimator of &nabla;<sub>&theta;</sub>U<sub>&theta;</sub> (i.e. ĝ) has high variance. If we can reduce its variance, then our estimate of gradient (&gcirc;) will be closer to the true gradient &nabla;<sub>&theta;</sub>U<sub>&theta;</sub>.

While the Monte-Carlo estimator of the gradient (&gcirc;) is unbiased, it exhibits high variance. As discussed below, there are a few ways of reducing variance without introducing bias: 1) using causality and 2) using a baseline.

## Actor-Critic Algorithm

One way to reduce variance is by taking advantage of causality: &gcirc; updates all the actions in a trajectory based upon total rewards and not the rewards to go. That is to say, future actions affect past rewards, which is not possible in our causal Universe. So we can make the gradient estimator more realistic by using rewards to go as shown in the below equation.

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;\approx&space;\hat{g}&space;=&space;\frac{1}{m}&space;\sum_{i=1}^{m}&space;\left(&space;\sum_{t=0}^{T}&space;\left(&space;\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\left(&space;\sum_{t^{'}=t}^{T}r(s_{t^{'}},a_{t^{'}})&space;\right)&space;\right)&space;\right)_i&space;\:&space;...&space;\,&space;Equation&space;\,&space;7" title="\nabla_{\theta}U_{\theta} \approx \hat{g} = \frac{1}{m} \sum_{i=1}^{m} \left( \sum_{t=0}^{T} \left( \nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \left( \sum_{t^{'}=t}^{T}r(s_{t^{'}},a_{t^{'}}) \right) \right) \right)_i \: ... \, Equation \, 7" />

Note that using the rewards to go instead of the total rewards still results in an unbiased estimator of &nabla;<sub>&theta;</sub>U<sub>&theta;</sub> because causality is handled in the expectation in Equation 3 using P<sub>&theta;</sub>(&tau;). Moreover, doing so reduces variance because the rewards to go expression has fewer terms (and thus lower uncertainty) than the total rewards expression.

An important aside to note is that the rewards to go is really an estimate of the the q-value of (s<sub>t</sub>, a<sub>t</sub>). This is because the q-value is defined as follows:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Q_{P_{\theta}}(s_{t},&space;a_{t})&space;=&space;r(s_t,&space;a_t)&space;&plus;&space;E_{\tau&space;\sim&space;P_{\theta}(\tau|s_t,&space;a_t)}\left&space;[\sum&space;_{t^{'}=t&plus;1}^T&space;r(s_{t^{'}},&space;a_{t^{'}})&space;\mid&space;s_t,&space;a_t&space;\right&space;]&space;\:&space;...&space;\,&space;Equation&space;\,&space;8" title="Q_{P_{\theta}}(s_{t}, a_{t}) = r(s_t, a_t) + E_{\tau \sim P_{\theta}(\tau|s_t, a_t)}\left [\sum _{t^{'}=t+1}^T r(s_{t^{'}}, a_{t^{'}}) \mid s_t, a_t \right ] \: ... \, Equation \, 8" />


And so, if the trajectory &tau; is sampled from P<sub>&theta;</sub>(&tau;), then the single-sample Monte-Carlo estimate of Q<sub>P<sub>&theta;</sub></sub>(s<sub>t</sub>, a<sub>t</sub>) is just:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Q_{P_{\theta}}(s_{t},&space;a_{t})&space;\approx&space;\hat{Q}_{P_{\theta}}(s_{t},&space;a_{t})&space;=&space;\sum_{t^{'}=t}^T&space;r(s_{t^{'}},&space;a_{t^{'}})&space;\:&space;...&space;\,&space;Equation&space;\,&space;9" title="Q_{P_{\theta}}(s_{t}, a_{t}) \approx \hat{Q}_{P_{\theta}}(s_{t}, a_{t}) = \sum_{t^{'}=t}^T r(s_{t^{'}}, a_{t^{'}}) \: ... \, Equation \, 9" />

As shown above, instead of using the Monte-Carlo estimator of the rewards to go as in Equation 7, we can use the Q-value estimator of the rewards to go. As a result, Equation 7 can be re-written as:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;\approx&space;\hat{g}&space;=&space;\frac{1}{m}&space;\sum_{i=1}^{m}&space;\left(&space;\sum_{t=0}^{T}&space;\left(&space;\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\,&space;\hat{Q}_{P_\theta}(s_t,&space;a_t)&space;\right)&space;\right)_i&space;\:&space;...&space;\,&space;Equation&space;\,&space;10" title="\nabla_{\theta}U_{\theta} \approx \hat{g} = \frac{1}{m} \sum_{i=1}^{m} \left( \sum_{t=0}^{T} \left( \nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \, \hat{Q}_{P_\theta}(s_t, a_t) \right) \right)_i \: ... \, Equation \, 10" />

If Qhat<sub>P<sub>&theta;</sub></sub>(s<sub>t</sub>, a<sub>t</sub>) is modeled using a neural network (parameterized by w), then we get:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\hat{Q}_{P_{\theta}}(s_{t},&space;a_{t})&space;=&space;\hat{Q}_{P_{\theta}}(s_{t},&space;a_{t},&space;w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;11" title="\hat{Q}_{P_{\theta}}(s_{t}, a_{t}) = \hat{Q}_{P_{\theta}}(s_{t}, a_{t}, w) \: ... \, Equation \, 11" />

Note that because the state-action space can be very high dimensional, it quickly runs into Bellman's curse of dimensionality; and thus, in most practical situations with complex state-transition dynamics, Qhat<sub>P<sub>&theta;</sub></sub>(s<sub>t</sub>, a<sub>t</sub>) is modeled using a neural network based function approximator.

Then Equation 10 can be re-written as:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;\approx&space;\hat{g}&space;=&space;\frac{1}{m}&space;\sum_{i=1}^{m}&space;\left(&space;\sum_{t=0}^{T}&space;\left(&space;\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\,&space;\hat{Q}_{P_\theta}(s_t,&space;a_t,&space;w)&space;\right)&space;\right)_i&space;\:&space;...&space;\,&space;Equation&space;\,&space;12" title="\nabla_{\theta}U_{\theta} \approx \hat{g} = \frac{1}{m} \sum_{i=1}^{m} \left( \sum_{t=0}^{T} \left( \nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \, \hat{Q}_{P_\theta}(s_t, a_t, w) \right) \right)_i \: ... \, Equation \, 12" />

Whereby, P<sub>&theta;</sub>(a<sub>t</sub> | s<sub>t</sub>) is the actor network that is parameterized by &theta; and Qhat<sub>P<sub>&theta;</sub></sub>(s<sub>t</sub>, a<sub>t</sub>, w) is the critic network that is parameterized by w. This is essentially what is known as the actor-critic algorithm.

For any visited state-action pair (s,a), the actor network is updated using Equation 6 (utilizing &gcirc; from Equation 12), and the critic network is typically updated using Temporal-Difference learning (due to its lower variance than Monte-Carlo learning) using the following update equation:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;w&space;\leftarrow&space;w&space;-&space;\beta&space;\nabla_{w}L(w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;13" title="w \leftarrow w - \beta \nabla_{w}L(w) \: ... \, Equation \, 13" />

Whereby the weight vector w is updated to reduce the loss L(w), which is defined as:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;L(w)&space;=&space;\frac{1}2{}(Q_{P_\theta}(s_t,a_t)&space;-&space;\hat{Q}_{P_\theta}(s_t,a_t,w))^2&space;\:&space;...&space;\,&space;Equation&space;\,&space;14" title="L(w) = \frac{1}2{}(Q_{P_\theta}(s_t,a_t) - \hat{Q}_{P_\theta}(s_t,a_t,w))^2 \: ... \, Equation \, 14" />

and using Q-learning (so that the critic is based of off an off-policy algorithm):

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Q_{P_\theta}(s_t,a_t)&space;\approx&space;r(s_t,a_t)&space;&plus;&space;\max_a\hat{Q}_{P_\theta}(s_{t&plus;1},a,w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;15" title="Q_{P_\theta}(s_t,a_t) \approx r(s_t,a_t) + \max_a\hat{Q}_{P_\theta}(s_{t+1},a,w) \: ... \, Equation \, 15" />

and so

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_wL(w)&space;=&space;-(r(s_t,a_t)&space;&plus;&space;\max_a\hat{Q}_{P_\theta}(s_{t&plus;1},a,w)&space;-&space;\hat{Q}_{P_\theta}(s_{t},a_{t},w))\nabla_w\hat{Q}_{P_\theta}(s_{t},a_{t},w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;16" title="\nabla_wL(w) = -(r(s_t,a_t) + \max_a\hat{Q}_{P_\theta}(s_{t+1},a,w) - \hat{Q}_{P_\theta}(s_{t},a_{t},w))\nabla_w\hat{Q}_{P_\theta}(s_{t},a_{t},w) \: ... \, Equation \, 16" />

whereby

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\delta_{TD&space;\;&space;Error}&space;=&space;r(s_t,a_t)&space;&plus;&space;\max_a\hat{Q}_{P_\theta}(s_{t&plus;1},a,w)&space;-&space;\hat{Q}_{P_\theta}(s_{t},a_{t},w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;17" title="\delta_{TD \; Error} = r(s_t,a_t) + \max_a\hat{Q}_{P_\theta}(s_{t+1},a,w) - \hat{Q}_{P_\theta}(s_{t},a_{t},w) \: ... \, Equation \, 17" />

This is the basics of the actor-critic algorithm. While there are many variants of it, as we will see below, this is the basic core of it.

## Advantage Actor-Critic Algorithm

In addition to using the rewards to go (due to causality), another approach to minimizing the variance of &gcirc; is by subtracting out a baseline b that is not dependent on &theta; or action a -- and this combined term is known as the Advantage function. It can be mathematically proved that such a transformation is not only unbiased, but it reduces variance. An intuitive explanation for why it reduces variance is because the term multiplying &nabla;<sub>&theta;</sub>log(P<sub>&theta;</sub>(a|s)) has smaller magnitude, which essentially reduces the variance of the overall expression.

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;\approx&space;\hat{g}&space;=&space;\frac{1}{m}&space;\sum_{i=1}^{m}&space;\left(&space;\sum_{t=0}^{T}&space;\left(&space;\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\,&space;\left(&space;\hat{Q}_{P_\theta}(s_t,&space;a_t,&space;w)&space;-&space;b\right)&space;\right)&space;\right)_i&space;\:&space;...&space;\,&space;Equation&space;\,&space;18" title="\nabla_{\theta}U_{\theta} \approx \hat{g} = \frac{1}{m} \sum_{i=1}^{m} \left( \sum_{t=0}^{T} \left( \nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \, \left( \hat{Q}_{P_\theta}(s_t, a_t, w) - b\right) \right) \right)_i \: ... \, Equation \, 18" />

There are many choices for the baseline b, and in theory, the optimal value of b can also be computed. However, in the interest of simplicity and to be intuitive, a commonly used baseline is the q-value averaged over all the actions, i.e. the state-value.

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;b&space;=&space;\hat{V}_{P_\theta}(s_t,&space;w)&space;=&space;E_{a_t&space;\sim&space;P_{\theta}(a_t|s_t)}&space;\left&space;[\hat{Q}_{P_\theta}(s_t,&space;a_t,&space;w)&space;\right&space;]&space;\:&space;...&space;\,&space;Equation&space;\,&space;19" title="b = \hat{V}_{P_\theta}(s_t, w) = E_{a_t \sim P_{\theta}(a_t|s_t)} \left [\hat{Q}_{P_\theta}(s_t, a_t, w) \right ] \: ... \, Equation \, 19" />

The Advantage function is then written as follows:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\hat{A}_{P_\theta}(s_t,&space;a_t,w)&space;=&space;\hat{Q}_{P_\theta}(s_t,&space;a_t,&space;w)&space;-&space;\hat{V}_{P_\theta}(s_t,&space;w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;20" title="\hat{A}_{P_\theta}(s_t, a_t,w) = \hat{Q}_{P_\theta}(s_t, a_t, w) - \hat{V}_{P_\theta}(s_t, w) \: ... \, Equation \, 20" />

The basic idea with using this advantage function is that actions with higher q-value than the average (i.e. state-value) are reinforced where as other actions are inhibited. This makes a lot more intuitive sense than the gradient equation used in the original Reinforce algorithm. And so it's not totally surprising that Mathematically it results in lower variance. Moreover, now the gradient is no longer dependent on the absolute value of the rewards.

One problem with the above Equation is that, in practice, it is very difficult to compute the above expectation -- especially for continuous actions or high dimensional action space. Hence, the state-value function is modeled with a separate neural network that is parameterized by w<sub>v</sub> as follows:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\hat{V}(s_t,&space;w_v)&space;\approx&space;\hat{V}_{P_\theta}(s_t,&space;w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;21" title="\hat{V}(s_t, w_v) \approx \hat{V}_{P_\theta}(s_t, w) \: ... \, Equation \, 21" />

The advantage function now becomes:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\hat{A}_{P_\theta}(s_t,&space;a_t,&space;w)&space;\approx&space;\hat{A}(s_t,&space;a_t,&space;w,&space;w_v)&space;=&space;\hat{Q}_{P_\theta}(s_t,&space;a_t,&space;w)&space;-&space;\hat{V}(s_t,&space;w_v)&space;\:&space;...&space;\,&space;Equation&space;\,&space;22" title="\hat{A}_{P_\theta}(s_t, a_t, w) \approx \hat{A}(s_t, a_t, w, w_v) = \hat{Q}_{P_\theta}(s_t, a_t, w) - \hat{V}(s_t, w_v) \: ... \, Equation \, 22" />

The issue with this advantage function is that it requires two separate neural networks. With some clever re-ordering, we can re-write the Advantage function using a single neural network. However, inorder to do so, let us first re-visit the above analysis. Basically, the ideal Advantage function we would like to have is:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;A_{P_\theta}(s_t,&space;a_t)&space;=&space;Q_{P_\theta}(s_t,&space;a_t)&space;-&space;V_{P_\theta}(s_t)&space;\:&space;...&space;\,&space;Equation&space;\,&space;23" title="A_{P_\theta}(s_t, a_t) = Q_{P_\theta}(s_t, a_t) - V_{P_\theta}(s_t) \: ... \, Equation \, 23" />

As defined in Equation 8 above, state-action value can be further simplified interms of the state-value function as:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Q_{P_{\theta}}(s_{t},&space;a_{t})&space;=&space;r(s_t,&space;a_t)&space;&plus;&space;E_{s_{t^{'}}&space;\sim&space;P_{\theta}(s_{t^{'}}|s_t,&space;a_t)}\left&space;[V_{P_{\theta}}(s_{t^{'}})&space;\mid&space;s_t,&space;a_t&space;\right&space;]&space;\:&space;...&space;\,&space;Equation&space;\,&space;24" title="Q_{P_{\theta}}(s_{t}, a_{t}) = r(s_t, a_t) + E_{s_{t^{'}} \sim P_{\theta}(s_{t^{'}}|s_t, a_t)}\left [V_{P_{\theta}}(s_{t^{'}}) \mid s_t, a_t \right ] \: ... \, Equation \, 24" />

The single-sample Monte-Carlo estimate of Q<sub>P<sub>&theta;</sub></sub>(s<sub>t</sub>, a<sub>t</sub>) as defined in the Equation above is:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Q_{P_{\theta}}(s_{t},&space;a_{t})&space;\approx&space;\hat{Q}_{P_{\theta}}(s_{t},&space;a_{t})&space;=&space;r(s_t,&space;a_t)&space;&plus;&space;V_{P_{\theta}}(s_{t^{'}})&space;\:&space;...&space;\,&space;Equation&space;\,&space;25" title="Q_{P_{\theta}}(s_{t}, a_{t}) \approx \hat{Q}_{P_{\theta}}(s_{t}, a_{t}) = r(s_t, a_t) + V_{P_{\theta}}(s_{t^{'}}) \: ... \, Equation \, 25" />

And so now we just need to represent the state-value function using a neural network parameterized by w<sub>v</sub> as follows:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;V_{P_{\theta}}(s_{t})&space;\approx&space;\hat{V}_{P_{\theta}}(s_{t},&space;w_v)&space;\:&space;...&space;\,&space;Equation&space;\,&space;26" title="V_{P_{\theta}}(s_{t}) \approx \hat{V}_{P_{\theta}}(s_{t}, w_v) \: ... \, Equation \, 26" />

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\hat{Q}_{P_{\theta}}(s_{t},&space;a_{t})&space;\approx&space;\hat{Q}_{P_{\theta}}(s_{t},&space;a_{t},&space;w_v)&space;=&space;r(s_t,&space;a_t)&space;&plus;&space;\hat{V}_{P_{\theta}}(s_{t^{'}},&space;w_v)&space;\:&space;...&space;\,&space;Equation&space;\,&space;27" title="\hat{Q}_{P_{\theta}}(s_{t}, a_{t}) \approx \hat{Q}_{P_{\theta}}(s_{t}, a_{t}, w_v) = r(s_t, a_t) + \hat{V}_{P_{\theta}}(s_{t^{'}}, w_v) \: ... \, Equation \, 27" />

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;A_{P_{\theta}}(s_{t},&space;a_{t})&space;\approx&space;\hat{A}_{P_{\theta}}(s_{t},&space;a_{t},&space;w_v)&space;=&space;r(s_t,&space;a_t)&space;&plus;&space;\hat{V}_{P_{\theta}}(s_{t^{'}},&space;w_v)&space;-&space;\hat{V}_{P_{\theta}}(s_t,&space;w_v)&space;\:&space;...&space;\,&space;Equation&space;\,&space;28" title="A_{P_{\theta}}(s_{t}, a_{t}) \approx \hat{A}_{P_{\theta}}(s_{t}, a_{t}, w_v) = r(s_t, a_t) + \hat{V}_{P_{\theta}}(s_{t^{'}}, w_v) - \hat{V}_{P_{\theta}}(s_t, w_v) \: ... \, Equation \, 28" />

And thus the Advantage function can now be represented using a single neural network parameterized with w<sub>v</sub>. Note with the above equation for the Advantage function, it is really just the one-step TD error (i.e. TD(0) error). Additionally, it is also possible to represent it using TD(&lambda;) error.

The gradient equation for Advantage Actor Critic is now going to be:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{\theta}U_{\theta}&space;\approx&space;\hat{g}&space;=&space;\frac{1}{m}&space;\sum_{i=1}^{m}&space;\left(&space;\sum_{t=0}^{T}&space;\left(&space;\nabla_{\theta}log(P_{\theta}(a_{t}|s_{t}))&space;\,&space;\hat{A}_{P_{\theta}}(s_{t},&space;a_{t},&space;w_v)&space;\right)&space;\right)_i&space;\:&space;...&space;\,&space;Equation&space;\,&space;29" title="\nabla_{\theta}U_{\theta} \approx \hat{g} = \frac{1}{m} \sum_{i=1}^{m} \left( \sum_{t=0}^{T} \left( \nabla_{\theta}log(P_{\theta}(a_{t}|s_{t})) \, \hat{A}_{P_{\theta}}(s_{t}, a_{t}, w_v) \right) \right)_i \: ... \, Equation \, 29" />

And this is going to be a much better estimator of the expected gradient (Equation 3), i.e. with lower variance and still be unbiased, even for m=1. As a result, the algorithm will learn much faster.

w<sub>v</sub> is updated as follows:

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;w_v&space;\leftarrow&space;w_v&space;-&space;\beta\nabla_{w_{v}}L(w_v)&space;\:&space;...&space;\,&space;Equation&space;\,&space;30" title="w_v \leftarrow w_v - \beta\nabla_{w_{v}}L(w_v) \: ... \, Equation \, 30" />

whereby using one-step TD learning (i.e. TD(0)):

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\nabla_{w_{v}}L(w_v)&space;=&space;-(r(s_t,&space;a_t)&space;&plus;&space;\hat{V}_{P_{\theta}}(s_{t^{'}},&space;w_v)&space;-&space;\hat{V}_{P_{\theta}}(s_t,&space;w_v))\nabla_{w_v}\hat{V}_{P_{\theta}}(s_t,&space;w_v)&space;=&space;-\hat{A}_{P_{\theta}}(s_t,a_t,w_v)&space;\nabla_{w_v}\hat{V}_{P_{\theta}}(s_t,&space;w_v)&space;\;&space;...&space;\,&space;Equation&space;\,&space;31" title="\nabla_{w_{v}}L(w_v) = -(r(s_t, a_t) + \hat{V}_{P_{\theta}}(s_{t^{'}}, w_v) - \hat{V}_{P_{\theta}}(s_t, w_v))\nabla_{w_v}\hat{V}_{P_{\theta}}(s_t, w_v) = -\hat{A}_{P_{\theta}}(s_t,a_t,w_v) \nabla_{w_v}\hat{V}_{P_{\theta}}(s_t, w_v) \; ... \, Equation \, 31" />

Using the gradient estimator from Equation 29, the weight update from Equation 30, and the remaining steps from the basic Reinforce algorithm results in what is known as the Advantage Actor-Critic algorithm.

To briefly summarize the above discussion, the main downside of the Reinforce algorithm is that the gradient estimator is based upon the Monte-Carlo estimator of the expected total reward from the initial state-action pair -- which while has low bias, it has high variance. By using causality and subtracting out a baseline from the Monte-Carlo estimator, we can reduce the variance. The variance is further reduced by using TD estimator of the expected total reward to go instead of Monte-Carlo estimator.

## Deterministic Policy Gradient (DPG) Algorithm

For stochastic policies in continuous environments, the actor outputs the mean and variance of a Gaussian distribution. And an action is sampled from this Gaussian distribution. For deterministic actions, while this approach still works as the network will learn to have very low variance, it involves complexity and computational burden that unnecessarily slows down the learning algorithm. To address these short comings, for deterministic actions, we can use what is known as the deterministic policy gradient.

In stochastic case, the policy gradient integrates over both state and action spaces, whereas in the deterministic case it only integrates over the state space. As a result, computing the deterministic policy gradient can potentially require fewer samples. But in order to fully explore the state space, the basic idea is to choose actions according to a stochastic behavior policy and learn about a deterministic target policy (i.e. needs to be an off-policy algorithm).

DPG is essentially a deterministic version of Actor-Critic algorithm. For a basic DPG algorithm, we have two neural networks, one network (parameterized by &theta;) is estimating the optimal target policy and the second network (parameterized by w) is estimating the action-value function corresponding to the target policy. The below equations formalize this.

As mentioned above, because the target policy is deterministic, the actor may not explore the state-space very well to find the optimal policy. To address it, we use a behavior policy (b(s<sub>t</sub>)) that is different from the target policy. It is basically the target policy with some additional noise. For simplicity, we will use a Normal distribution as our noise source. But note that this term is like a hyper parameter, and in the below implementation for the Reacher environment, a different noise process is used.

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Target&space;\:&space;Policy&space;=&space;\mu_{\theta}(s_t)&space;\:&space;...&space;\,&space;Equation&space;\,&space;32a" title="Target \: Policy = \mu_{\theta}(s_t) \: ... \, Equation \, 32a" />

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;b(s_t)&space;=&space;\mu_{\theta}(s_t)&space;&plus;&space;\mathit{N}(0,I)&space;\:&space;...&space;\,&space;Equation&space;\,&space;32b" title="b(s_t) = \mu_{\theta}(s_t) + \mathit{N}(0,I) \: ... \, Equation \, 32b" />

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;a_t&space;\sim&space;b(s_t)&space;\:&space;...&space;\,&space;Equation&space;\,&space;32c" title="a_t \sim b(s_t) \: ... \, Equation \, 32c" />

<img src="https://latex.codecogs.com/png.latex?\fn_cm&space;Q_{\theta}(s_t,a_t)&space;\approx&space;\hat{Q}_{\theta}(s_t,a_t,w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;33" title="Q_{\theta}(s_t,a_t) \approx \hat{Q}_{\theta}(s_t,a_t,w) \: ... \, Equation \, 33" />

**Deterministic Policy Gradient Update:**<br>
1. Actor network is updated as follows:

    <img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\theta&space;\leftarrow&space;\theta&space;&plus;&space;\alpha_{\theta}\nabla_{\theta}\hat{Q}_{\theta}(s_t,\mu_{\theta}(s_t),w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;34" title="\theta \leftarrow \theta + \alpha_{\theta}\nabla_{\theta}\hat{Q}_{\theta}(s_t,\mu_{\theta}(s_t),w) \: ... \, Equation \, 34" />

    which by chain rule, it becomes:

    <img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\theta&space;\leftarrow&space;\theta&space;&plus;&space;\alpha_{\theta}\nabla_{a}\hat{Q}_{\theta}(s_t,a,w)\nabla_{\theta}\mu_{\theta}(s_t)&space;\:&space;...&space;\,&space;Equation&space;\,&space;35" title="\theta \leftarrow \theta + \alpha_{\theta}\nabla_{a}\hat{Q}_{\theta}(s_t,a,w)\nabla_{\theta}\mu_{\theta}(s_t) \: ... \, Equation \, 35" />

2. The critic network is updated as follows:

    The TD error is given by:

    <img src="https://latex.codecogs.com/png.latex?\fn_cm&space;\delta_t&space;=&space;r(s_t,a_t)&space;&plus;&space;\hat{Q}_{\theta}(s_{t&plus;1},&space;\mu_{\theta}(s_{t&plus;1}),w)&space;-&space;\hat{Q}_{\theta}(s_t,&space;a_t&space;\sim&space;b(s_t),w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;36" title="\delta_t = r(s_t,a_t) + \hat{Q}_{\theta}(s_{t+1}, \mu_{\theta}(s_{t+1}),w) - \hat{Q}_{\theta}(s_t, a_t \sim b(s_t),w) \: ... \, Equation \, 36" />

    and the weight update is:

    <img src="https://latex.codecogs.com/png.latex?\fn_cm&space;w&space;\leftarrow&space;w&space;&plus;&space;\alpha_w\delta_t\nabla_w\hat{Q}_{\theta}(s_t,&space;a_t&space;\sim&space;b(s_t),w)&space;\:&space;...&space;\,&space;Equation&space;\,&space;37" title="w \leftarrow w + \alpha_w\delta_t\nabla_w\hat{Q}_{\theta}(s_t, a_t \sim b(s_t),w) \: ... \, Equation \, 37" />

To reiterate, in order to properly balance exploration-exploitation tradeoff, while the target policy &mu; is deterministic, the behavior policy is stochastic. So this is an off-policy version of the DPG algorithm. While stochastic off-policy actor-critic algorithms typically use importance sampling for both the actor and the critic, because the deterministic policy gradient removes expectation over the actions, and given the state transition dynamics are same for both the target and behavior policies as they operate in the same environment, importance sampling ratio is not needed. So we can avoid having to use importance sampling in the actor, and with same reasoning, we avoid using importance sampling in the critic [2]. For those who are wondering, similar reasoning applies as to why we don't use importance sampling with Q-learning.

## Deep Deterministic Policy Gradient (DDPG) Algorithm
DDPG is basically DPG with a few training changes adopted from the DQN architecture.

One challenge when using neural networks for reinforcement learning is that most optimization algorithms assume the samples are independently and identically distributed. Obviously this assumption doesn't hold true because the samples are generated by exploring sequentially in an environment. Because DDPG is an off policy algorithm, we can use the replay buffer (a finite sized cache) as in DQN to address this issue. At each timestep the actor and critic are updated by sampling a minibatch uniformly from the buffer [2].

For the critic, since the network being updated is also used in calculating the target, this can potentially lead to training instabilities for highly nonlinear function approximators like neural networks. One solution to address this is using a separate target network, as with DQN [2]. Given the target values are determined using both the critic and actor networks, we create a copy of both of these networks and soft update their weights to the respective learned networks. [Please refer to my github code for details.](https://github.com/gtg162y/DRLND/blob/master/P2_Continuous_Actions/Continuous_Control_UdacityWorkspace.ipynb)

---------------------------------------------------------------
## DDPG Implementation for Reacher Environment

Having now seen some of the commonly used policy gradient algorithms, we can now get to my implementation for the Udacity's Reacher project. In this environment, a double-jointed arm (acrobot) can move to target locations (i.e.  where the balloons are). A reward of +0.1 is provided for each step that the agent's hand is in the goal location. Thus, the goal of the agent is to maintain its position at the target location for as many time steps as possible. As the balloon moves, the two joints at adjusted to track the balloon. So this is a classical robotics project, and using model-free reinforcement learning, the agent will learn the optimal policy. In particular, the method used is the deep deterministic policy gradient method (DDPG). The observation space consists of 33 variables corresponding to position, rotation, velocity, and angular velocities of the arm. Each action is a vector with four numbers, corresponding to torque applicable to two joints. Every entry in the action vector is a number between -1 and 1.

The Reacher environment used contains 20 identical agents, each with its own copy of the environment. In order to be considered solved, the agents must get an average score of +30 (over 100 consecutive episodes, and over all 20 agents). In particular, after each episode, we add up the rewards that each agent received (without discounting), to get a score for each agent. This yields 20 (potentially different) scores. We then take the average of these 20 scores.
This yields an average score for each episode (where the average is over all 20 agents).
The environment is considered solved, when the average (over 100 episodes) of those average scores is at least +30.

The DDPG algorithm uses 4 separate neural networks. One to learn the policy, another to learn the value function, another for the target value function, and one for the target action in the target action-value function network. As discussed earlier, we use a separate target network instead of the local network so as to prevent any instabilities in learning when the TD target is dependent on the local network. The weights in the target network are updated very slowly, like 0.1% towards the local network at every time step. Slowly changing the target network does slow down the learning rate, but it helps with the learning algorithm's stability [2]. 

To speed up the learning process, since all 20 agents are experiencing at the same time, the input data stream is pretty large. To adequately take advantage of this, I perform network parameter update 4 times at each iteration. The reason this works without using an importance sampling ratio is because the target policy for the actor is deterministic, as well as the target value for the critic is also deterministic. Additionally, the critic network's gradients are clipped to 1. This prevents the critic network from changing too fast. Moreover, I initialized the target networks to have the same (random) weights as the networks being learned. Additionally, instead of Normal distribution, Ornstein-Uhlenbeck noise process is used to generate the behavior policy for better exploration. All these things together allowed the agent to achieve the learning objective in just over 100 episodes. [Please refer to my github code for details.](https://github.com/gtg162y/DRLND/blob/master/P2_Continuous_Actions/Continuous_Control_UdacityWorkspace.ipynb)

Below is the learning performance of the algorithm averaged over all the 20 agents.

![Scores][image2]

In terms of hyperparameters used, the learning rate for both the actor and critic network was 1e-4, no regularization was used as the networks were fairly small. I gradually decayed the exploration probability to get the optimal policy. Ornstein-Uhlenbeck noise process used a mean value of 0 and sigma of 0.2. The actor network was built using a three layer neural network (with 256 neurons in the first layer, 128 neurons in the second layer, and 4 neurons in the final output layer). The critic network was also built using three layers and similar number of neurons as the actor network, except that actions were concatenated to the output of the first layer and the final layer had a single output neuron. For faster learning, elu non-linearity was used for both networks.

In terms of methods to further improve the agent's performance, a couple things on my to do list are: 1) train the agent using Prioritized Experience Replay as well as 2) use the Proximal Policy Optimization algorithm.

**References:**<br>
1: UC Berkeley CS294 Lectures (http://rail.eecs.berkeley.edu/deeprlcourse/)<br>
2. DDPG paper (https://arxiv.org/pdf/1509.02971.pdf)


[comment]: # (Equations generated using: https://stackoverflow.com/questions/11256433/how-to-show-math-equations-in-general-githubs-markdownnot-githubs-blog,
https://www.codecogs.com/latex/eqneditor.php,
http://mathurl.com/)
