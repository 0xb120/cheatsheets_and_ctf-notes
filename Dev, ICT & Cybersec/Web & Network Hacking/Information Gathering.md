Is the phase where every possible information about the target is gather from the attacker in order to spread the attack surface.
Think of information gathering as creating a map of the terrain, while enumeration is exploring specific locations on that map in detail. Information gathering tells you what might exist, while enumeration confirms what actually exists and provides technical details that can be used for exploitation later.

Usually, general scanning phases looks like that:
1. Massive recon and target identification
2. Fingerprinting
3. Target Enumeration
	- Port Scans
	- OS Fingerprint
	- Version Scans
4. [Vulnerability Scanning](Vulnerability%20Scanning.md)

## Information gathering (Reconnaissance)

The initial, broader phase where you collect general information about the target. Focuses on publicly available information.

- [Passive information gathering (OSINT)](Passive%20information%20gathering%20(OSINT).md)
- [Active information gathering](Active%20information%20gathering.md)
- Physical Tracking
	- [Using SMS for Location](../../Readwise/Articles/d8rh8r%20-%20Using%20SMS%20for%20Location.md)
	- Tracking location using *mobile ad data* [^ad-data]
	- SS7 Attack [^ss7-first-attack][^ss7-veritasium]

[^ad-data]: [How They Tracked the Trump Shooter's Phone](../../Readwise/Articles/Seytonic%20-%20How%20They%20Tracked%20the%20Trump%20Shooter's%20Phone.md), Seytonic
[^ss7-first-attack]: [FIsher Price “My First SS7 Attack”](../../Readwise/Articles/d8rh8r%20-%20FIsher%20Price%20“My%20First%20SS7%20Attack”.md), d8rh8r
[^ss7-veritasium]: [I Hacked My Friend's Phone to Show How Easy It Is](../../Readwise/Articles/Veritasium%20-%20I%20Hacked%20My%20Friend's%20Phone%20to%20Show%20How%20Easy%20It%20Is.md), Veritasium

## Enumeration

Every further confirmed information, such as software versions, open ports, etc. gathered from specific checks on every single service or identified target.

- [Services list](../Services/Services%20list.md)
	- [HTTP Recon and Enumeration](HTTP%20Recon%20and%20Enumeration.md)
- [Active Directory Enumeration](Active%20Directory%20Enumeration.md)
- [Internal information gathering](Internal%20information%20gathering.md)