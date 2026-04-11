# Nim 语言新的性能测试

> 原文：https://www.cnblogs.com/tansm/p/16149673.html

今天将 性能测试网站： [benchmarks game](https://benchmarksgame-team.pages.debian.net/benchmarksgame/index.html) 上一个关于 [n-body](https://benchmarksgame-team.pages.debian.net/benchmarksgame/performance/nbody.html) 的题目改成 nim 1.6.4 语言来编写。

注意，我是基于 java 的版本来写的，没有像  c++ 那样的版本使用 SIMD 技术，因为我认为，我纯粹是为了看看编译器，在执行普通的方法，其效率如何。

```text
import std/math

const
  PI = 3.141592653589793
  SOLAR_MASS = 4 * PI * PI
  DAYS_PER_YEAR = 365.24
  LENGTH = 5

type
  Body = ref object
    x  : float64
    y  : float64
    z  : float64
    vx : float64
    vy : float64
    vz : float64
    mass : float64

  NNodySystem = ref object of RootObj
    bodies : seq[Body]

proc offsetMomentum(this : var Body, px : float64, py : float64, pz : float64) =
  this.vx = -px / SOLAR_MASS
  this.vy = -py / SOLAR_MASS
  this.vz = -pz / SOLAR_MASS

proc jupiter() : Body =
  Body(
    x : 4.84143144246472090e+00,
    y : -1.16032004402742839e+00,
    z : -1.03622044471123109e-01,
    vx : 1.66007664274403694e-03 * DAYS_PER_YEAR,
    vy : 7.69901118419740425e-03 * DAYS_PER_YEAR,
    vz : -6.90460016972063023e-05 * DAYS_PER_YEAR,
    mass : 9.54791938424326609e-04 * SOLAR_MASS
  )

proc saturn() : Body =
  Body(
    x : 8.34336671824457987e+00,
    y : 4.12479856412430479e+00,
    z : -4.03523417114321381e-01,
    vx : -2.76742510726862411e-03 * DAYS_PER_YEAR,
    vy : 4.99852801234917238e-03 * DAYS_PER_YEAR,
    vz : 2.30417297573763929e-05 * DAYS_PER_YEAR,
    mass : 2.85885980666130812e-04 * SOLAR_MASS
  )

proc uranus() : Body =
  Body(
    x : 1.28943695621391310e+01,
    y : -1.51111514016986312e+01,
    z : -2.23307578892655734e-01,
    vx: 2.96460137564761618e-03 * DAYS_PER_YEAR,
    vy: 2.37847173959480950e-03 * DAYS_PER_YEAR,
    vz: -2.96589568540237556e-05 * DAYS_PER_YEAR,
    mass : 4.36624404335156298e-05 * SOLAR_MASS
  )

proc neptune() : Body =
  Body(
    x : 1.53796971148509165e+01,
    y : -2.59193146099879641e+01,
    z : 1.79258772950371181e-01,
    vx : 2.68067772490389322e-03 * DAYS_PER_YEAR,
    vy : 1.62824170038242295e-03 * DAYS_PER_YEAR,
    vz : -9.51592254519715870e-05 * DAYS_PER_YEAR,
    mass : 5.15138902046611451e-05 * SOLAR_MASS
  )

proc sun() : Body =
  Body(
    mass : SOLAR_MASS
  )

proc newBodySystem() : NNodySystem =
  result = NNodySystem(
    bodies : @[sun(),jupiter(),saturn(),uranus(),neptune()]
  )
  var px,py,pz = 0.0
  var bodies = result.bodies
  for i in 0 ..< LENGTH:
    px += bodies[i].vx * bodies[i].mass
    py += bodies[i].vy * bodies[i].mass
    pz += bodies[i].vz * bodies[i].mass
  bodies[0].offsetMomentum(px,py,pz)

proc advance(this: NNodySystem, dt : float64) =
  let b = this.bodies
  for i in 0 ..< LENGTH - 1:
    var iBody = b[i]
    let iMass = iBody.mass
    let ix = iBody.x
    let iy = iBody.y
    let iz = iBody.z
    for j in i+1 ..< LENGTH:
      var jBody = b[j]
      let dx = ix - jBody.x
      let dy = iy - jBody.y
      let dz = iz - jBody.z
      let dSquared = dx * dx + dy * dy + dz * dz
      let distance = sqrt(dSquared)
      let mag = dt / (dSquared * distance)
      let jMass = jBody.mass
      iBody.vx -= dx * jMass * mag
      iBody.vy -= dy * jMass * mag
      iBody.vz -= dz * jMass * mag
      jBody.vx += dx * iMass * mag
      jBody.vy += dy * iMass * mag
      jBody.vz += dz * iMass * mag
  for i in 0 ..< LENGTH:
    var body = b[i]
    body.x += dt * body.vx
    body.y += dt * body.vy
    body.z += dt * body.vz

proc energy(this : NNodySystem) : float64 =
  var dx,dy,dz,distance,e : float64
  for i, iBody in this.bodies:
    e += 0.5 * iBody.mass * (iBody.vx * iBody.vx + iBody.vy * iBody.vy + iBody.vz * iBody.vz)
    for j in i + 1 ..< this.bodies.len:
      let jBody = this.bodies[j]
      dx = iBody.x - jBody.x
      dy = iBody.y - jBody.y
      dz = iBody.z - jBody.z
      distance = sqrt(dx * dx + dy * dy + dz * dz)
      e -= iBody.mass * jBody.mass / distance

  return e

proc main(n : int32) =
  let bodies = newBodySystem()
  echo bodies.energy()
  for i in 0 ..< n:
    bodies.advance(0.01)
  echo bodies.energy()

main(50000000)
```

 

使用 release 编译

```text
nim c -d:release .\NBodyTestX64.nim
```

在我的机器上，java 版本接近 5秒，而 nim 的版本，需要 5.5 秒。

 我也实验了 Kotlin native 1.6.20，仍然需要8~9秒。
