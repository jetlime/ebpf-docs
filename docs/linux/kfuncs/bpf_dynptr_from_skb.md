---
title: "KFunc 'bpf_dynptr_from_skb'"
description: "This page documents the 'bpf_dynptr_from_skb' eBPF kfunc, including its definition, usage, program types that can use it, and examples."
---
# KFunc `bpf_dynptr_from_skb`

<!-- [FEATURE_TAG](bpf_dynptr_from_skb) -->
[:octicons-tag-24: v6.4](https://github.com/torvalds/linux/commit/b5964b968ac64c2ec2debee7518499113b27c34e)
<!-- [/FEATURE_TAG] -->

Get dynptrs whose underlying pointer points to a skb.

## Definition

For bpf program types that don't support writes on skb data, the dynptr is read-only ([`bpf_dynptr_write()`](../helper-function/bpf_dynptr_write.md) will return an error)

For reads and writes through the [`bpf_dynptr_read()`](../helper-function/bpf_dynptr_read.md) and [`bpf_dynptr_write()`](../helper-function/bpf_dynptr_write.md) interfaces, reading and writing from/to data in the head as well as from/to non-linear paged buffers is supported. Data slices through the bpf_dynptr_data API are not supported; instead [`bpf_dynptr_slice()`](bpf_dynptr_slice.md) and [`bpf_dynptr_slice_rdwr()`](bpf_dynptr_slice_rdwr.md) should be used.

**Signature**

<!-- [KFUNC_DEF] -->
`#!c int bpf_dynptr_from_skb(struct __sk_buff *s, u64 flags, struct bpf_dynptr *ptr__uninit)`
<!-- [/KFUNC_DEF] -->

## Usage

The dynptr acts on skb data. skb dynptrs have two main benefits. One is that they allow operations on sizes that are not statically known at compile-time (for example variable-sized accesses). Another is that parsing the packet data through dynptrs (instead of through direct access of skb->data and skb->data_end) can be more ergonomic and less brittle (does not need manual if checking for being within bounds of data_end).

### Program types

The following program types can make use of this kfunc:

<!-- [KFUNC_PROG_REF] -->
- [`BPF_PROG_TYPE_CGROUP_DEVICE`](../program-type/BPF_PROG_TYPE_CGROUP_DEVICE.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/67666479edf1e2b732f4d0ac797885e859a78de4) - 
- [`BPF_PROG_TYPE_CGROUP_SKB`](../program-type/BPF_PROG_TYPE_CGROUP_SKB.md)
- [`BPF_PROG_TYPE_CGROUP_SOCK`](../program-type/BPF_PROG_TYPE_CGROUP_SOCK.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/67666479edf1e2b732f4d0ac797885e859a78de4) - 
- [`BPF_PROG_TYPE_CGROUP_SOCKOPT`](../program-type/BPF_PROG_TYPE_CGROUP_SOCKOPT.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/67666479edf1e2b732f4d0ac797885e859a78de4) - 
- [`BPF_PROG_TYPE_CGROUP_SOCK_ADDR`](../program-type/BPF_PROG_TYPE_CGROUP_SOCK_ADDR.md) [:octicons-tag-24: v6.7](https://github.com/torvalds/linux/commit/53e380d21441909b12b6e0782b77187ae4b971c4) - 
- [`BPF_PROG_TYPE_CGROUP_SYSCTL`](../program-type/BPF_PROG_TYPE_CGROUP_SYSCTL.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/67666479edf1e2b732f4d0ac797885e859a78de4) - 
- [`BPF_PROG_TYPE_LSM`](../program-type/BPF_PROG_TYPE_LSM.md)
- [`BPF_PROG_TYPE_LWT_IN`](../program-type/BPF_PROG_TYPE_LWT_IN.md)
- [`BPF_PROG_TYPE_LWT_OUT`](../program-type/BPF_PROG_TYPE_LWT_OUT.md)
- [`BPF_PROG_TYPE_LWT_SEG6LOCAL`](../program-type/BPF_PROG_TYPE_LWT_SEG6LOCAL.md)
- [`BPF_PROG_TYPE_LWT_XMIT`](../program-type/BPF_PROG_TYPE_LWT_XMIT.md)
- [`BPF_PROG_TYPE_NETFILTER`](../program-type/BPF_PROG_TYPE_NETFILTER.md)
- [`BPF_PROG_TYPE_PERF_EVENT`](../program-type/BPF_PROG_TYPE_PERF_EVENT.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/bc638d8cb5be813d4eeb9f63cce52caaa18f3960) - 
- [`BPF_PROG_TYPE_SCHED_ACT`](../program-type/BPF_PROG_TYPE_SCHED_ACT.md)
- [`BPF_PROG_TYPE_SCHED_CLS`](../program-type/BPF_PROG_TYPE_SCHED_CLS.md)
- [`BPF_PROG_TYPE_SK_SKB`](../program-type/BPF_PROG_TYPE_SK_SKB.md)
- [`BPF_PROG_TYPE_SOCKET_FILTER`](../program-type/BPF_PROG_TYPE_SOCKET_FILTER.md)
- [`BPF_PROG_TYPE_SOCK_OPS`](../program-type/BPF_PROG_TYPE_SOCK_OPS.md) [:octicons-tag-24: v6.15](https://github.com/torvalds/linux/commit/59422464266f8baa091edcb3779f0955a21abf00) - 
- [`BPF_PROG_TYPE_TRACEPOINT`](../program-type/BPF_PROG_TYPE_TRACEPOINT.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/bc638d8cb5be813d4eeb9f63cce52caaa18f3960) - 
- [`BPF_PROG_TYPE_TRACING`](../program-type/BPF_PROG_TYPE_TRACING.md)
<!-- [/KFUNC_PROG_REF] -->

### Example

??? example "L4LB no inline dynptr"
    ```c
    // SPDX-License-Identifier: GPL-2.0
    // Copyright (c) 2017 Facebook
    #include <stddef.h>
    #include <stdbool.h>
    #include <string.h>
    #include <linux/pkt_cls.h>
    #include <linux/bpf.h>
    #include <linux/in.h>
    #include <linux/if_ether.h>
    #include <linux/ip.h>
    #include <linux/ipv6.h>
    #include <linux/icmp.h>
    #include <linux/icmpv6.h>
    #include <linux/tcp.h>
    #include <linux/udp.h>
    #include <bpf/bpf_helpers.h>
    #include "test_iptunnel_common.h"
    #include <bpf/bpf_endian.h>

    #include "bpf_kfuncs.h"

    static __always_inline __u32 rol32(__u32 word, unsigned int shift)
    {
        return (word << shift) | (word >> ((-shift) & 31));
    }

    /* copy paste of jhash from kernel sources to make sure llvm
    * can compile it into valid sequence of bpf instructions
    */
    #define __jhash_mix(a, b, c)			\
    {						\
        a -= c;  a ^= rol32(c, 4);  c += b;	\
        b -= a;  b ^= rol32(a, 6);  a += c;	\
        c -= b;  c ^= rol32(b, 8);  b += a;	\
        a -= c;  a ^= rol32(c, 16); c += b;	\
        b -= a;  b ^= rol32(a, 19); a += c;	\
        c -= b;  c ^= rol32(b, 4);  b += a;	\
    }

    #define __jhash_final(a, b, c)			\
    {						\
        c ^= b; c -= rol32(b, 14);		\
        a ^= c; a -= rol32(c, 11);		\
        b ^= a; b -= rol32(a, 25);		\
        c ^= b; c -= rol32(b, 16);		\
        a ^= c; a -= rol32(c, 4);		\
        b ^= a; b -= rol32(a, 14);		\
        c ^= b; c -= rol32(b, 24);		\
    }

    #define JHASH_INITVAL		0xdeadbeef

    typedef unsigned int u32;

    static __noinline u32 jhash(const void *key, u32 length, u32 initval)
    {
        u32 a, b, c;
        const unsigned char *k = key;

        a = b = c = JHASH_INITVAL + length + initval;

        while (length > 12) {
            a += *(u32 *)(k);
            b += *(u32 *)(k + 4);
            c += *(u32 *)(k + 8);
            __jhash_mix(a, b, c);
            length -= 12;
            k += 12;
        }
        switch (length) {
        case 12: c += (u32)k[11]<<24;
        case 11: c += (u32)k[10]<<16;
        case 10: c += (u32)k[9]<<8;
        case 9:  c += k[8];
        case 8:  b += (u32)k[7]<<24;
        case 7:  b += (u32)k[6]<<16;
        case 6:  b += (u32)k[5]<<8;
        case 5:  b += k[4];
        case 4:  a += (u32)k[3]<<24;
        case 3:  a += (u32)k[2]<<16;
        case 2:  a += (u32)k[1]<<8;
        case 1:  a += k[0];
            __jhash_final(a, b, c);
        case 0: /* Nothing left to add */
            break;
        }

        return c;
    }

    static __noinline u32 __jhash_nwords(u32 a, u32 b, u32 c, u32 initval)
    {
        a += initval;
        b += initval;
        c += initval;
        __jhash_final(a, b, c);
        return c;
    }

    static __noinline u32 jhash_2words(u32 a, u32 b, u32 initval)
    {
        return __jhash_nwords(a, b, 0, initval + JHASH_INITVAL + (2 << 2));
    }

    #define PCKT_FRAGMENTED 65343
    #define IPV4_HDR_LEN_NO_OPT 20
    #define IPV4_PLUS_ICMP_HDR 28
    #define IPV6_PLUS_ICMP_HDR 48
    #define RING_SIZE 2
    #define MAX_VIPS 12
    #define MAX_REALS 5
    #define CTL_MAP_SIZE 16
    #define CH_RINGS_SIZE (MAX_VIPS * RING_SIZE)
    #define F_IPV6 (1 << 0)
    #define F_HASH_NO_SRC_PORT (1 << 0)
    #define F_ICMP (1 << 0)
    #define F_SYN_SET (1 << 1)

    struct packet_description {
        union {
            __be32 src;
            __be32 srcv6[4];
        };
        union {
            __be32 dst;
            __be32 dstv6[4];
        };
        union {
            __u32 ports;
            __u16 port16[2];
        };
        __u8 proto;
        __u8 flags;
    };

    struct ctl_value {
        union {
            __u64 value;
            __u32 ifindex;
            __u8 mac[6];
        };
    };

    struct vip_meta {
        __u32 flags;
        __u32 vip_num;
    };

    struct real_definition {
        union {
            __be32 dst;
            __be32 dstv6[4];
        };
        __u8 flags;
    };

    struct vip_stats {
        __u64 bytes;
        __u64 pkts;
    };

    struct eth_hdr {
        unsigned char eth_dest[ETH_ALEN];
        unsigned char eth_source[ETH_ALEN];
        unsigned short eth_proto;
    };

    struct {
        __uint(type, BPF_MAP_TYPE_HASH);
        __uint(max_entries, MAX_VIPS);
        __type(key, struct vip);
        __type(value, struct vip_meta);
    } vip_map SEC(".maps");

    struct {
        __uint(type, BPF_MAP_TYPE_ARRAY);
        __uint(max_entries, CH_RINGS_SIZE);
        __type(key, __u32);
        __type(value, __u32);
    } ch_rings SEC(".maps");

    struct {
        __uint(type, BPF_MAP_TYPE_ARRAY);
        __uint(max_entries, MAX_REALS);
        __type(key, __u32);
        __type(value, struct real_definition);
    } reals SEC(".maps");

    struct {
        __uint(type, BPF_MAP_TYPE_PERCPU_ARRAY);
        __uint(max_entries, MAX_VIPS);
        __type(key, __u32);
        __type(value, struct vip_stats);
    } stats SEC(".maps");

    struct {
        __uint(type, BPF_MAP_TYPE_ARRAY);
        __uint(max_entries, CTL_MAP_SIZE);
        __type(key, __u32);
        __type(value, struct ctl_value);
    } ctl_array SEC(".maps");

    static __noinline __u32 get_packet_hash(struct packet_description *pckt, bool ipv6)
    {
        if (ipv6)
            return jhash_2words(jhash(pckt->srcv6, 16, MAX_VIPS),
                        pckt->ports, CH_RINGS_SIZE);
        else
            return jhash_2words(pckt->src, pckt->ports, CH_RINGS_SIZE);
    }

    static __noinline bool get_packet_dst(struct real_definition **real,
                        struct packet_description *pckt,
                        struct vip_meta *vip_info,
                        bool is_ipv6)
    {
        __u32 hash = get_packet_hash(pckt, is_ipv6);
        __u32 key = RING_SIZE * vip_info->vip_num + hash % RING_SIZE;
        __u32 *real_pos;

        if (hash != 0x358459b7 /* jhash of ipv4 packet */  &&
            hash != 0x2f4bc6bb /* jhash of ipv6 packet */)
            return false;

        real_pos = bpf_map_lookup_elem(&ch_rings, &key);
        if (!real_pos)
            return false;
        key = *real_pos;
        *real = bpf_map_lookup_elem(&reals, &key);
        if (!(*real))
            return false;
        return true;
    }

    static __noinline int parse_icmpv6(struct bpf_dynptr *skb_ptr, __u64 off,
                    struct packet_description *pckt)
    {
        __u8 buffer[sizeof(struct ipv6hdr)] = {};
        struct icmp6hdr *icmp_hdr;
        struct ipv6hdr *ip6h;

        icmp_hdr = bpf_dynptr_slice(skb_ptr, off, buffer, sizeof(buffer));
        if (!icmp_hdr)
            return TC_ACT_SHOT;

        if (icmp_hdr->icmp6_type != ICMPV6_PKT_TOOBIG)
            return TC_ACT_OK;
        off += sizeof(struct icmp6hdr);
        ip6h = bpf_dynptr_slice(skb_ptr, off, buffer, sizeof(buffer));
        if (!ip6h)
            return TC_ACT_SHOT;
        pckt->proto = ip6h->nexthdr;
        pckt->flags |= F_ICMP;
        memcpy(pckt->srcv6, ip6h->daddr.s6_addr32, 16);
        memcpy(pckt->dstv6, ip6h->saddr.s6_addr32, 16);
        return TC_ACT_UNSPEC;
    }

    static __noinline int parse_icmp(struct bpf_dynptr *skb_ptr, __u64 off,
                    struct packet_description *pckt)
    {
        __u8 buffer_icmp[sizeof(struct iphdr)] = {};
        __u8 buffer_ip[sizeof(struct iphdr)] = {};
        struct icmphdr *icmp_hdr;
        struct iphdr *iph;

        icmp_hdr = bpf_dynptr_slice(skb_ptr, off, buffer_icmp, sizeof(buffer_icmp));
        if (!icmp_hdr)
            return TC_ACT_SHOT;
        if (icmp_hdr->type != ICMP_DEST_UNREACH ||
            icmp_hdr->code != ICMP_FRAG_NEEDED)
            return TC_ACT_OK;
        off += sizeof(struct icmphdr);
        iph = bpf_dynptr_slice(skb_ptr, off, buffer_ip, sizeof(buffer_ip));
        if (!iph || iph->ihl != 5)
            return TC_ACT_SHOT;
        pckt->proto = iph->protocol;
        pckt->flags |= F_ICMP;
        pckt->src = iph->daddr;
        pckt->dst = iph->saddr;
        return TC_ACT_UNSPEC;
    }

    static __noinline bool parse_udp(struct bpf_dynptr *skb_ptr, __u64 off,
                    struct packet_description *pckt)
    {
        __u8 buffer[sizeof(struct udphdr)] = {};
        struct udphdr *udp;

        udp = bpf_dynptr_slice(skb_ptr, off, buffer, sizeof(buffer));
        if (!udp)
            return false;

        if (!(pckt->flags & F_ICMP)) {
            pckt->port16[0] = udp->source;
            pckt->port16[1] = udp->dest;
        } else {
            pckt->port16[0] = udp->dest;
            pckt->port16[1] = udp->source;
        }
        return true;
    }

    static __noinline bool parse_tcp(struct bpf_dynptr *skb_ptr, __u64 off,
                    struct packet_description *pckt)
    {
        __u8 buffer[sizeof(struct tcphdr)] = {};
        struct tcphdr *tcp;

        tcp = bpf_dynptr_slice(skb_ptr, off, buffer, sizeof(buffer));
        if (!tcp)
            return false;

        if (tcp->syn)
            pckt->flags |= F_SYN_SET;

        if (!(pckt->flags & F_ICMP)) {
            pckt->port16[0] = tcp->source;
            pckt->port16[1] = tcp->dest;
        } else {
            pckt->port16[0] = tcp->dest;
            pckt->port16[1] = tcp->source;
        }
        return true;
    }

    static __noinline int process_packet(struct bpf_dynptr *skb_ptr,
                        struct eth_hdr *eth, __u64 off,
                        bool is_ipv6, struct __sk_buff *skb)
    {
        struct packet_description pckt = {};
        struct bpf_tunnel_key tkey = {};
        struct vip_stats *data_stats;
        struct real_definition *dst;
        struct vip_meta *vip_info;
        struct ctl_value *cval;
        __u32 v4_intf_pos = 1;
        __u32 v6_intf_pos = 2;
        struct ipv6hdr *ip6h;
        struct vip vip = {};
        struct iphdr *iph;
        int tun_flag = 0;
        __u16 pkt_bytes;
        __u64 iph_len;
        __u32 ifindex;
        __u8 protocol;
        __u32 vip_num;
        int action;

        tkey.tunnel_ttl = 64;
        if (is_ipv6) {
            __u8 buffer[sizeof(struct ipv6hdr)] = {};

            ip6h = bpf_dynptr_slice(skb_ptr, off, buffer, sizeof(buffer));
            if (!ip6h)
                return TC_ACT_SHOT;

            iph_len = sizeof(struct ipv6hdr);
            protocol = ip6h->nexthdr;
            pckt.proto = protocol;
            pkt_bytes = bpf_ntohs(ip6h->payload_len);
            off += iph_len;
            if (protocol == IPPROTO_FRAGMENT) {
                return TC_ACT_SHOT;
            } else if (protocol == IPPROTO_ICMPV6) {
                action = parse_icmpv6(skb_ptr, off, &pckt);
                if (action >= 0)
                    return action;
                off += IPV6_PLUS_ICMP_HDR;
            } else {
                memcpy(pckt.srcv6, ip6h->saddr.s6_addr32, 16);
                memcpy(pckt.dstv6, ip6h->daddr.s6_addr32, 16);
            }
        } else {
            __u8 buffer[sizeof(struct iphdr)] = {};

            iph = bpf_dynptr_slice(skb_ptr, off, buffer, sizeof(buffer));
            if (!iph || iph->ihl != 5)
                return TC_ACT_SHOT;

            protocol = iph->protocol;
            pckt.proto = protocol;
            pkt_bytes = bpf_ntohs(iph->tot_len);
            off += IPV4_HDR_LEN_NO_OPT;

            if (iph->frag_off & PCKT_FRAGMENTED)
                return TC_ACT_SHOT;
            if (protocol == IPPROTO_ICMP) {
                action = parse_icmp(skb_ptr, off, &pckt);
                if (action >= 0)
                    return action;
                off += IPV4_PLUS_ICMP_HDR;
            } else {
                pckt.src = iph->saddr;
                pckt.dst = iph->daddr;
            }
        }
        protocol = pckt.proto;

        if (protocol == IPPROTO_TCP) {
            if (!parse_tcp(skb_ptr, off, &pckt))
                return TC_ACT_SHOT;
        } else if (protocol == IPPROTO_UDP) {
            if (!parse_udp(skb_ptr, off, &pckt))
                return TC_ACT_SHOT;
        } else {
            return TC_ACT_SHOT;
        }

        if (is_ipv6)
            memcpy(vip.daddr.v6, pckt.dstv6, 16);
        else
            vip.daddr.v4 = pckt.dst;

        vip.dport = pckt.port16[1];
        vip.protocol = pckt.proto;
        vip_info = bpf_map_lookup_elem(&vip_map, &vip);
        if (!vip_info) {
            vip.dport = 0;
            vip_info = bpf_map_lookup_elem(&vip_map, &vip);
            if (!vip_info)
                return TC_ACT_SHOT;
            pckt.port16[1] = 0;
        }

        if (vip_info->flags & F_HASH_NO_SRC_PORT)
            pckt.port16[0] = 0;

        if (!get_packet_dst(&dst, &pckt, vip_info, is_ipv6))
            return TC_ACT_SHOT;

        if (dst->flags & F_IPV6) {
            cval = bpf_map_lookup_elem(&ctl_array, &v6_intf_pos);
            if (!cval)
                return TC_ACT_SHOT;
            ifindex = cval->ifindex;
            memcpy(tkey.remote_ipv6, dst->dstv6, 16);
            tun_flag = BPF_F_TUNINFO_IPV6;
        } else {
            cval = bpf_map_lookup_elem(&ctl_array, &v4_intf_pos);
            if (!cval)
                return TC_ACT_SHOT;
            ifindex = cval->ifindex;
            tkey.remote_ipv4 = dst->dst;
        }
        vip_num = vip_info->vip_num;
        data_stats = bpf_map_lookup_elem(&stats, &vip_num);
        if (!data_stats)
            return TC_ACT_SHOT;
        data_stats->pkts++;
        data_stats->bytes += pkt_bytes;
        bpf_skb_set_tunnel_key(skb, &tkey, sizeof(tkey), tun_flag);
        *(u32 *)eth->eth_dest = tkey.remote_ipv4;
        return bpf_redirect(ifindex, 0);
    }

    SEC("tc")
    int balancer_ingress(struct __sk_buff *ctx)
    {
        __u8 buffer[sizeof(struct eth_hdr)] = {};
        struct bpf_dynptr ptr;
        struct eth_hdr *eth;
        __u32 eth_proto;
        __u32 nh_off;
        int err;

        nh_off = sizeof(struct eth_hdr);

        bpf_dynptr_from_skb(ctx, 0, &ptr);
        eth = bpf_dynptr_slice_rdwr(&ptr, 0, buffer, sizeof(buffer));
        if (!eth)
            return TC_ACT_SHOT;
        eth_proto = eth->eth_proto;
        if (eth_proto == bpf_htons(ETH_P_IP))
            err = process_packet(&ptr, eth, nh_off, false, ctx);
        else if (eth_proto == bpf_htons(ETH_P_IPV6))
            err = process_packet(&ptr, eth, nh_off, true, ctx);
        else
            return TC_ACT_SHOT;

        if (eth == buffer)
            bpf_dynptr_write(&ptr, 0, buffer, sizeof(buffer), 0);

        return err;
    }

    char _license[] SEC("license") = "GPL";
    ```
