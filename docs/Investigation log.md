# Investigation log

Newest first. One entry per finding: what was looked at, how, and what it showed. This is the raw material for report section 2 (how the incident was discovered), so record the method too, not just the result.

<!-- Template:
## YYYY-MM-DD HH:MM Name
- Looked at:
- How (filter, tool, query):
- Result:
- Next:
-->

## 2026-11-07 12:39 CTI sources: first pass on iodine and DNS tunnelling
- Looked at: three pages, read through a summarising fetch (not yet read in full by Rick).
  - MITRE ATT&CK T1572 Protocol Tunneling: https://attack.mitre.org/techniques/T1572/
  - MITRE ATT&CK T1071.004 Application Layer Protocol: DNS: https://attack.mitre.org/techniques/T1071/004/
  - iodine repository: https://github.com/yarrick/iodine
- Result:
  - T1572 lists the **Cutting Edge** campaign as using **Iodine** to tunnel IPv4 over DNS. Other DNS tunnel users listed include Chimera, Cobalt Strike, Heyoka Backdoor and Uroburos.
  - T1071.004 describes DNS tunnelling as hiding commands and results in DNS traffic, names groups such as OilRig, FIN7 and APT41, and gives detection advice: long or frequent subdomains, encoded subdomains, high query volume, tools such as iodine and dnscat2. This is the CTI base for our features.
  - iodine README: tunnels IPv4 over DNS. **NULL** gives the most downstream bandwidth. Upstream data is gzipped and encoded in the query name (Base32, Base64, Base64u or **Base128 with high-byte characters**, autodetected). The server needs a real domain with an NS record delegated to the iodined host. Mentions protocol version `00000502`.
- What this supports: NULL records and non-ASCII bytes in names (Base128) match what we see. The `VACK` handshake itself is **not yet confirmed**: the fetched pages did not show the protocol specification. Check the repository's `doc/` folder.
- Next: read the `doc/` folder of the iodine repository for the handshake; read the OilRig and Cutting Edge ATT&CK pages for the attacker's order of steps.

## 2026-11-07 12:14 Wire-level checks in Wireshark and tshark: name length, payload, response sizes, replayed burst
- Looked at: packets 489 and 490 (first tunnel query and its answer), name lengths on the wire, response frame sizes, the two bursts, NXDOMAIN and unanswered queries.
- How: Wireshark GUI on `case1.pcap` (UTC time display), `tshark` with `dns.qry.type == 10`, `dns.id == ...`, `-T fields -e dns.qry.name.len -e frame.len`.
- Result:
  - **Name length trap:** longest tunnel query name is **252 bytes on the wire** (DNS limit 255). The 774 in `dns.log` is Zeek's escaped rendering (each non-printable byte written as `\xNN`, four characters). Benign names are plain ASCII, so the log and the wire agree there (max 103). A length feature built on the log text inflates the tunnel. Slice 3 must choose the length definition deliberately, and the report must say which.
  - **Payload only in the PCAP:** the answer to the first query (frame 490) holds a 9-byte NULL record, `56 41 43 4b 44 03 c5 e9 01`. The first four bytes read `VACK`, which matches iodine's version acknowledgement as I remember its protocol (to confirm against iodine documentation in the CTI step). Zeek shows it as `<unknown type=10>`.
  - **Response sizes:** tunnel responses median 179 bytes, max 1,512 (424 responses); benign median 162, max 427. The typical difference is small, the tail is large. Responses are normally bigger than queries in DNS, so size alone is a moderate signal.
  - **Replayed burst:** the second burst (18:50 to 18:51) sends the same 222 names as the first (18:04). Risk for slice 4: a time split puts identical tunnel names on both sides and inflates the evaluation. Also each name appears twice in the data.
  - **NXDOMAIN does not mark the tunnel:** all 11 NXDOMAIN rows are benign; every tunnel row that got an answer got NOERROR. The NXDOMAIN ratio feature in the task division will not discriminate here.
  - **Unanswered queries:** 21 queries have no response in the PCAP, which explains the 21 empty `rcode_name` values (20 tunnel, 1 benign).
  - **Volume does not find it:** the benign client peaks at 551 to 598 queries per minute, the tunnel at 221. A plain query-rate rule would flag the wrong traffic. The second tunnel burst overlaps a busy benign period (18:50 to 18:52).
  - Captured MAC addresses start with `08:00:27`, the VirtualBox vendor prefix, which supports the lab inference.
- Next: tell slice 3 and slice 4 about the length definition and the replayed burst when the schema and split are discussed.

## 2026-11-07 11:39 Clients, resolvers and source ports
- Looked at: which client talks to which resolver, and the source ports used.
- How: crosstab of `id.orig_h` by `id.resp_h`, and distinct `id.orig_p` per client, in pandas on `dns.log`.
- Result:
  - Two separate networks, no crossover. `192.168.179.21` uses resolver `192.168.179.1` (3,201 queries; a home router, the log contains `fritz.box`). `10.0.2.30` uses resolver `10.0.2.20` (444 queries). `10.0.2.x` is the VirtualBox default range, so the tunnel was probably generated in a lab (inference, not verified).
  - The benign client spreads 3,201 queries over 244 parent domains and 2,895 distinct source ports. The tunnel client sends all 444 queries from **one source port**.
- Consequences:
  - **Dataset limitation:** host identity (IP, resolver, source port) separates the classes perfectly. These must not be model features, or the evaluation looks perfect for the wrong reason. Goes into the data description and the discussion (lab versus real traffic).
  - **Fixed source port:** an IoC for Appendix A, but a weak characteristic for the "hard for the attacker to change" question, since a different tool or configuration randomises the port. The strong characteristics are those inherent to carrying data in names (long names, many unique subdomains, NULL records).
- Next: tell slice 3 the do-not-use list when the schema is agreed.

## 2026-11-07 11:05 Tunnel found in dns.log, confirmed in PCAP, ground truth v1 written
- Looked at: `data/raw/dns.log` (3,645 rows, 2023-01-07 17:35 to 19:10 UTC) and `data/raw/case1.pcap` (7,269 packets, all DNS).
- How: `notebooks/01_discovery.ipynb`. Parent domains ranked by unique names, mean and max name length, client count. Crosstab of domain suffix, record type and client. PCAP parsed with scapy and compared with the log.
- Result:
  - Two clients: `192.168.179.21` (3,201 rows, normal browsing) and `10.0.2.30` (444 rows).
  - **Tunnel domain `pirate.sea`**: 444 queries, 222 unique names, mean length 89, max 774. The only parent domain with record type NULL. Longest benign name is 103 characters.
  - **Client `10.0.2.30`** sends exactly the 444 `pirate.sea` queries and nothing else. Domain, record type NULL and client coincide on every row.
  - **Two bursts of 222 queries**: 18:04:36 to about 18:05 and 18:50 to 18:51. The second burst repeats the same 222 names as the first.
  - 20 of the 444 tunnel queries have no response in the log.
  - **PCAP agrees with the log**: 3,645 DNS queries and 3,624 responses. `pirate.sea` accounts for 444 queries (all type 10 NULL, all from `10.0.2.30`) and 424 responses. Responses are larger (mean 208 bytes, max 1,512) than the queries (mean 118 bytes, max 323).
  - Names look like **iodine** (hypothesis, not yet checked against CTI): first names `vaaaa...` (version), `l...` (login), `r...` probes with non-ASCII bytes, and readable German and French sentences in the codec check names. iodine uses NULL queries.
- Label rule: `tunnel` = query name under `pirate.sea`, else `benign`. 444 tunnel, 3,201 benign, tunnel share 12.2%. Written to `data/labels.csv`, `row_id` = 0-based position among data rows of `dns.log`.
- Next: check the iodine hypothesis against CTI and the iodine protocol description. Wireshark pass on the PCAP for screenshots and filters (`dns.qry.type == 10`, `dns.qry.name contains "pirate.sea"`). Explain the repeated burst (replay, or two sessions). Tell Samir and slice 3 the row id convention.
