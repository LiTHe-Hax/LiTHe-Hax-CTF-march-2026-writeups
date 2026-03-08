The point which is closest to all the drones combined is at the center of gravity (tyngdpunkt),
which is the average coordinate of all the points.

From the first 2 drone scans, it is possible to calculate the velocity of the drones.

The position of each drone is (its position at t=0) + (its velocity)*t

Now add up the square of the distance to each drone from the center of gravity at several timesteps, 
the timestep where the sum is lowest is the optimal time, and the (x, y) is the center of gravity.

Solution script:
```
import numpy as np

from pwn import *


HOST = "127.0.0.1"
PORT = 80
HAS_SSL = False
r = remote(HOST, PORT, ssl=HAS_SSL)


def get_best_pos_P(t):
    best_x = 1/NUM_DRONES * sum(o_poss[i][0] + o_v[i][0]*t for i in range(len(o_v)))
    best_y = 1/NUM_DRONES * sum(o_poss[i][1] + o_v[i][1]*t for i in range(len(o_v)))
    return (best_x, best_y)


def get_best_time():
    samples = 10
    times = list(range(samples))
    dists = np.empty(samples)
    for t in times:
        dists[t] = get_tot_dist(t)
    return (times, dists)


def get_tot_dist(t):
    tyngd_p = get_best_pos_P(t)
    tot_dist = sum(
        (tyngd_p[0] - (o_poss[i][0] + o_v[i][0] * t)) ** 2
        + (tyngd_p[1] - (o_poss[i][1] + o_v[i][1])) ** 2
        for i in range(NUM_DRONES))
    return tot_dist


def get_opt():
    times = get_best_time()
    best_dist = min(times[1])
    if len([i for i in times[1] if i == best_dist]) != 1:
        return "dup"
    return list(times[1]).index(best_dist), best_dist


def calc_ans():
    best_time = get_opt()
    best_P = get_best_pos_P(best_time[0])
    ans = int(np.floor(best_time[0] * best_P[0] * best_P[1]))
    return ans


def get_poses():
    r.sendline(b"") # With 'press enter'
    r.recvuntil(b"[")
    p1 = r.recvuntil(b"]").decode()
    p1 = eval("[" + p1)

    r.recvuntil(b"[")
    p2 = r.recvuntil(b"]").decode()
    p2 = eval("[" + p2)
    
    r.recvuntil(b":")

    return p1, p2
    

def get_o_v(poses):
    p1, p2 = poses
    o_p = p1
    ov = [(p2[i][0] - p1[i][0],
           p2[i][1] - p1[i][1])
          for i in range(NUM_DRONES)]
    return o_p, ov


def send_ans_get_flag(ans):
    ans = str(ans).encode("utf-8")
    r.sendline(ans)
    return r.recvall().decode()


if __name__ == "__main__":
    poses = get_poses()
    NUM_DRONES = len(poses[0])
    o_poss, o_v = get_o_v(poses)
    ans = calc_ans()
    print(ans)
    flag = send_ans_get_flag(ans)
    print(flag)
```
