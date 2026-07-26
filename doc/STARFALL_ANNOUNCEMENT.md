> [!NOTE]
> Please be aware that this incident is **ONGOING**, and the report is by no means complete.  
> Details may change or become available in the future.

The following report details events pertaining to a denial-of-service campaign directed at Starfall.  
It contains an important security advisory. Please share with others you think may be impacted, or are at risk.

## Table of Contents

1. [The Timeline](#the-timeline)
2. [The Evidence](#the-evidence)
    1. [Firewall Logs](#firewall-logs)
    2. [A Lesson in Operations Security](#a-lesson-in-operations-security)
3. [The Exploit](#the-exploit)
    1. [Scope](#scope)
    2. [Mitigation](#mitigation)
4. [Conclusion](#conclusion)
5. [TLDR](#tldr)

## The Timeline

For clarity's sake, all timestamps are in EST unless written otherwise.

1. July 20th, 4:30 AM - Sign-in is made mandatory to access parts of the website
   We made the unfortunate decision to lock down the website after receiving multiple reports
   of users being harassed for having an account on Starfall. We still regret doing this, but we
   are working on an opt-in visibility feature, so the site can remain open.

2. July 20th, 8:30 PM - Reports of game server connection issues  
   Players who were online at the time were complaining of connection issues in-game.  
   This caused us to investigate what the issue could be.

3. July 21st, 12:48 AM - Sign-in requirement temporarily reverted as part of investigation
   While the change should not have caused the reported issues, the requirement was temporarily
   removed for diagnostic measures.

4. July 21st, 1:00 AM - Foul play identified  
   A test was conducted where all network traffic, excluding traffic from one individual, was blocked
   from the game server.

    The test was a success, and we became aware that the reported issue was caused by an individual
    performing a denial-of-service attack on the game servers.

5. July 21st, 2:00 AM - Attacker identity verified  
   After some digging into how the exploit works, we decided to figure out who this attacker was.
   Search engines revealed that the offending IP address belonged to XlXi, owner and operator of the
   unfinished revival VirtuBrick. This point is expanded on in [The Evidence](#the-evidence).

6. July 21st, 3:00 AM - Sign-in requirement reinstated
   Following the confirmation that the requirement was not the cause of the issue, it was reinstated.

7. July 21st, 4:00 AM - Packet captures finalized, initial firewall rules created  
   At this point, we now have ~200 MB of raw captured traffic from the game server VM, all confirming
   that XlXi is performing this attack.

## The Evidence

We can't say with certainty _why_ XlXi is doing this. His cycle of concern-trolling and batshit insane claims makes it
seriously difficult to tell when he is being serious or stroking his ego. We are well aware at this point that XlXi has
been targeting Sylvessa since she was 13 back in Finobe, so this appears to be a personal vendetta rather than any _actual_
concern. Please keep his bias in mind when he tries to slander Starfall or Sylvessa.

To most of you, this is not a surprise. But, others might be left questioning these claims. So, here's the evidence that XlXi is the individual behind this.

### Firewall Logs

The firewall is configured to `DROP` all traffic originating from `104.61.230.122`, and to create a log entry.  
Each line represents a packet that was discarded.

```
103 4 tap103i0-IN 21/Jul/2026:07:12:02 -0400 DROP: IN=fwbr103i0 OUT=fwbr103i0 PHYSIN=fwln103i0 PHYSOUT=tap103i0 MAC=bc:24:11:15:67:67:bc:f8:7e:8e:2a:ee:08:00 SRC=104.61.230.122 DST=108.18.94.49 LEN=53 TOS=0x00 PREC=0x00 TTL=54 ID=11418 DF PROTO=UDP SPT=49795 DPT=23700 LEN=33
103 4 tap103i0-IN 21/Jul/2026:07:12:02 -0400 DROP: IN=fwbr103i0 OUT=fwbr103i0 PHYSIN=fwln103i0 PHYSOUT=tap103i0 MAC=bc:24:11:15:67:67:bc:f8:7e:8e:2a:ee:08:00 SRC=104.61.230.122 DST=108.18.94.49 LEN=53 TOS=0x00 PREC=0x00 TTL=54 ID=58561 DF PROTO=UDP SPT=38870 DPT=23701 LEN=33
103 4 tap103i0-IN 21/Jul/2026:07:12:02 -0400 DROP: IN=fwbr103i0 OUT=fwbr103i0 PHYSIN=fwln103i0 PHYSOUT=tap103i0 MAC=bc:24:11:15:67:67:bc:f8:7e:8e:2a:ee:08:00 SRC=104.61.230.122 DST=108.18.94.49 LEN=53 TOS=0x00 PREC=0x00 TTL=54 ID=5911 DF PROTO=UDP SPT=36436 DPT=23702 LEN=33
103 4 tap103i0-IN 21/Jul/2026:07:12:02 -0400 DROP: IN=fwbr103i0 OUT=fwbr103i0 PHYSIN=fwln103i0 PHYSOUT=tap103i0 MAC=bc:24:11:15:67:67:bc:f8:7e:8e:2a:ee:08:00 SRC=104.61.230.122 DST=108.18.94.49 LEN=53 TOS=0x00 PREC=0x00 TTL=54 ID=31920 DF PROTO=UDP SPT=48928 DPT=23703 LEN=33
103 4 tap103i0-IN 21/Jul/2026:07:12:02 -0400 DROP: IN=fwbr103i0 OUT=fwbr103i0 PHYSIN=fwln103i0 PHYSOUT=tap103i0 MAC=bc:24:11:15:67:67:bc:f8:7e:8e:2a:ee:08:00 SRC=104.61.230.122 DST=108.18.94.49 LEN=53 TOS=0x00 PREC=0x00 TTL=54 ID=5295 DF PROTO=UDP SPT=46811 DPT=23705 LEN=33
```

Here's a breakdown of what these fields mean:

- `SRC=`: The source IP. Where the traffic is coming from.
- `DST=`: The destination IP. Where the traffic is going.
- `PROTO=`: Protocol. Not relevant here, but Starfall game servers only use `UDP`.
- `SPT=`: The source port. Not relevant in this breakdown, but listed for clarity.
- `DPT=`: The destination port. This is used to determine what service on the IP the traffic should be routed to.  
  Starfall uses port ranges `23700` to `23799`. If `DPT` is within that range, the traffic is for Starfall.

Now that we understand the logs, it shows that XlXi is sending data to Starfall game servers in bursts.  
By itself, this can be harmless. But when you associate when this traffic is detected, with when game servers crash, it becomes apparent that XlXi is conducting a denial-of-service attack. Unfortunately this isn't provable without our packet logs, but we cannot release those until a fix is public. We seriously do not want
to release these logs until we are sure a fix is public, and widely adopted.

### A Lesson in Operations Security

XlXi has a history of poor decisions like this. Previously, he organized an IP logging campaign on Tadah, on the same IP address directly
associated with him at the time. Now, he's done something similar to Starfall.

[Dropping the IP address into Shodan](https://www.shodan.io/host/104.61.230.122), reveals that it exposes a web server.  
The TLS certificate this web server defaults to is bound to the domain `virtubrick.local`. An unfinished revival that XlXi owns and operates.  
This effectively proves that the malicious traffic we have identified, is coming directly from the network XlXi operates.

Not enough for you? Add the following to your `hosts` file, and navigate to [www.virtubrick.local](https://www.virtubrick.local/) to see for yourself!

```
104.61.230.122 www.virtubrick.local
```

- [Archive 1](https://web.archive.org/web/20260721152713/https://www.shodan.io/host/104.61.230.122)
- [Archive 2](http://archive.today/2026.07.21-153011/https://www.shodan.io/host/104.61.230.122)

## The Exploit

While we are not currently releasing extended details on this exploit, we have fully reverse-engineered it, and understand how and why it works.
A proper fix to this vulnerability is in development, and will be published as **open-source software**, along with a proof-of-concept exploit.

### Scope

Latest testing reveals client versions `0.78.0.701` to `0.360.1.252096` are at risk, but this range is not definitive.  
As far as we know, this exploit works on clients from 2012 all the way up to 2018. **All currently public revivals are affected.**

### Mitigation

Since XlXi has stopped these attacks after being privately called out, mitigations _might_ not be required.  
Though, we do urge system administrators to **block** all traffic from his IP address.  
This obviously isn't foolproof, but it will give you some heads-up if, or when he attempts another attack.

Again, an open-source fix will be released when we can guarantee it is effective.

## Conclusion

That pretty much wraps it up for the exploit, at least. There's a huge amount of disinformation about Starfall and its members coming from XlXi that we'll have
to cover some other time. While we are working on that, I personally hope this report is enough justification to finally cut this guy off, if you haven't
already. Don't trust people who would rather commit actual felonies than talk things out, no matter what side they're on.

## TLDR

Because I know some of you hate reading.

- XlXi has been conducting a denial-of-service attack that uses a vulnerability in the client itself.
- It has been used on Starfall **and** Hexagon.
- The exploit works on clients ranging from 2012 all the way up to 2018.
- Revival operators are urged to apply firewall rules blocking his traffic, until the team at Starfall releases an open-source fix.
- Please be wary of his conduct and understand that he is not a trustworthy individual.
