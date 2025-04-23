# Introduction on Networked Physics

## Challenge

- same input, different output on different clients.
- network latency & jitter.

## Determinism

### what is 'determinism'
> Given same input, the final state is always the same, on any device, any platform, any time.
> 
> By https://www.ggpo.net/


### how to achieve 'determinism'
- use fixed point math
- do not rely on local time or local framerate, but tick
- ensure iteration order (e.g., dictionary iter order is not guaranteed)


### why common physics engines are not deterministic?
- they use floating point math rather than fixed point, because
  - floating point math is fast enough on morden computers
  - floating point math has much larger range of values
- the tool chains dealing with collider data, mesh… does not support fixed math.
- even with determinism (including your gameplay framework), we still have big problem to solve: latency.


## Latency & jitter

The basic idea against network latency and jitter is
- interpolation
- extrapolation
- client simulation + rollback

NO one-fit-all solution.

## Real world solutions
- UE4/UE5
- Rocket League
- WatchDog 2
- CSGO

## Other learning materials
- [Gaffer on Games](https://gafferongames.com/categories/networked-physics/)