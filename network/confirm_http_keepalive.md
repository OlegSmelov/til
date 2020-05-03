# Checking if HTTP Keep-Alive is working

Let's say you have a webserver that talks to another service through HTTP, e.g. Elasticsearch
(TCP port `9200`). If you're not sure if TCP connections are reused, you run `tcpdump` and watch its
output:

```sh
sudo tcpdump -n -i lo "tcp and (src port 9200 or dst port 9200)"
```

In this example, let's make two requests. Note the client ports (`55942` and `55946`) are
different, which means Keep-Alive is not working:

```
13:52:03.289485 IP 127.0.0.1.55942 > 127.0.0.1.9200: Flags [S], seq 3193513107, win 65495, options [mss 65495,sackOK,TS val 3422687451 ecr 0,nop,wscale 7], length 0
13:52:03.289498 IP 127.0.0.1.9200 > 127.0.0.1.55942: Flags [S.], seq 3639792603, ack 3193513108, win 65483, options [mss 65495,sackOK,TS val 3422687451 ecr 3422687451,nop,wscale 7], length 0
13:52:03.289507 IP 127.0.0.1.55942 > 127.0.0.1.9200: Flags [.], ack 1, win 512, options [nop,nop,TS val 3422687451 ecr 3422687451], length 0
13:52:03.289537 IP 127.0.0.1.55942 > 127.0.0.1.9200: Flags [P.], seq 1:79, ack 1, win 512, options [nop,nop,TS val 3422687452 ecr 3422687451], length 78
13:52:03.291481 IP 127.0.0.1.9200 > 127.0.0.1.55942: Flags [P.], seq 1:416, ack 79, win 512, options [nop,nop,TS val 3422687453 ecr 3422687452], length 415
13:52:03.291493 IP 127.0.0.1.55942 > 127.0.0.1.9200: Flags [.], ack 416, win 509, options [nop,nop,TS val 3422687453 ecr 3422687453], length 0
13:52:03.291607 IP 127.0.0.1.55942 > 127.0.0.1.9200: Flags [F.], seq 79, ack 416, win 512, options [nop,nop,TS val 3422687454 ecr 3422687453], length 0
13:52:03.291773 IP 127.0.0.1.9200 > 127.0.0.1.55942: Flags [F.], seq 416, ack 80, win 512, options [nop,nop,TS val 3422687454 ecr 3422687454], length 0
13:52:03.291786 IP 127.0.0.1.55942 > 127.0.0.1.9200: Flags [.], ack 417, win 512, options [nop,nop,TS val 3422687454 ecr 3422687454], length 0

# ...

13:52:39.582430 IP 127.0.0.1.55946 > 127.0.0.1.9200: Flags [S], seq 3392553636, win 65495, options [mss 65495,sackOK,TS val 3422723744 ecr 0,nop,wscale 7], length 0
13:52:39.582442 IP 127.0.0.1.9200 > 127.0.0.1.55946: Flags [S.], seq 4125370754, ack 3392553637, win 65483, options [mss 65495,sackOK,TS val 3422723744 ecr 3422723744,nop,wscale 7], length 0
13:52:39.582451 IP 127.0.0.1.55946 > 127.0.0.1.9200: Flags [.], ack 1, win 512, options [nop,nop,TS val 3422723744 ecr 3422723744], length 0
13:52:39.582481 IP 127.0.0.1.55946 > 127.0.0.1.9200: Flags [P.], seq 1:79, ack 1, win 512, options [nop,nop,TS val 3422723744 ecr 3422723744], length 78
13:52:39.582485 IP 127.0.0.1.9200 > 127.0.0.1.55946: Flags [.], ack 79, win 511, options [nop,nop,TS val 3422723744 ecr 3422723744], length 0
13:52:39.584474 IP 127.0.0.1.9200 > 127.0.0.1.55946: Flags [P.], seq 1:416, ack 79, win 512, options [nop,nop,TS val 3422723746 ecr 3422723744], length 415
13:52:39.584482 IP 127.0.0.1.55946 > 127.0.0.1.9200: Flags [.], ack 416, win 509, options [nop,nop,TS val 3422723746 ecr 3422723746], length 0
13:52:39.584604 IP 127.0.0.1.55946 > 127.0.0.1.9200: Flags [F.], seq 79, ack 416, win 512, options [nop,nop,TS val 3422723747 ecr 3422723746], length 0
13:52:39.584779 IP 127.0.0.1.9200 > 127.0.0.1.55946: Flags [F.], seq 416, ack 80, win 512, options [nop,nop,TS val 3422723747 ecr 3422723747], length 0
13:52:39.584787 IP 127.0.0.1.55946 > 127.0.0.1.9200: Flags [.], ack 417, win 512, options [nop,nop,TS val 3422723747 ecr 3422723747], length 0
```

When Keep-Alive is working, you'll see that client port doesn't change (`56030`):

```
14:03:36.626180 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [S], seq 1571642888, win 65495, options [mss 65495,sackOK,TS val 3423380788 ecr 0,nop,wscale 7], length 0
14:03:36.626193 IP 127.0.0.1.9200 > 127.0.0.1.56030: Flags [S.], seq 1666808550, ack 1571642889, win 65483, options [mss 65495,sackOK,TS val 3423380788 ecr 3423380788,nop,wscale 7], length 0
14:03:36.626203 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [.], ack 1, win 512, options [nop,nop,TS val 3423380788 ecr 3423380788], length 0
14:03:36.626233 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [P.], seq 1:79, ack 1, win 512, options [nop,nop,TS val 3423380788 ecr 3423380788], length 78
14:03:36.626237 IP 127.0.0.1.9200 > 127.0.0.1.56030: Flags [.], ack 79, win 511, options [nop,nop,TS val 3423380788 ecr 3423380788], length 0
14:03:36.627695 IP 127.0.0.1.9200 > 127.0.0.1.56030: Flags [P.], seq 1:416, ack 79, win 512, options [nop,nop,TS val 3423380790 ecr 3423380788], length 415
14:03:36.627705 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [.], ack 416, win 509, options [nop,nop,TS val 3423380790 ecr 3423380790], length 0
14:03:36.627861 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [P.], seq 79:157, ack 416, win 512, options [nop,nop,TS val 3423380790 ecr 3423380790], length 78
14:03:36.627871 IP 127.0.0.1.9200 > 127.0.0.1.56030: Flags [.], ack 157, win 512, options [nop,nop,TS val 3423380790 ecr 3423380790], length 0
14:03:36.628069 IP 127.0.0.1.9200 > 127.0.0.1.56030: Flags [P.], seq 416:831, ack 157, win 512, options [nop,nop,TS val 3423380790 ecr 3423380790], length 415
14:03:36.628075 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [.], ack 831, win 509, options [nop,nop,TS val 3423380790 ecr 3423380790], length 0
14:03:36.628116 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [F.], seq 157, ack 831, win 512, options [nop,nop,TS val 3423380790 ecr 3423380790], length 0
14:03:36.628248 IP 127.0.0.1.9200 > 127.0.0.1.56030: Flags [F.], seq 831, ack 158, win 512, options [nop,nop,TS val 3423380790 ecr 3423380790], length 0
14:03:36.628258 IP 127.0.0.1.56030 > 127.0.0.1.9200: Flags [.], ack 832, win 512, options [nop,nop,TS val 3423380790 ecr 3423380790], length 0
```

Keep in mind that connections are not kept open indefinitely, so you'll need to make the successive
requests quickly enough not to trigger a Keep-Alive timeout.
