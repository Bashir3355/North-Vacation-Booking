# Learning Practice: Black-Box Testing, Part 1

**Application:** North Vacation Booking  
**Testing method:** Black-box testing — equivalence partitioning  
**Deliverable:** Test design based only on the provided requirements (no source code or test execution)

# 1. Purpose and requirements

North Vacation Booking allows guests to reserve vacation rentals. A valid reservation has **1–31 nights**, **1–12 guests**, check-in before checkout, and **every reserved night available**. Checkout is not a reserved night. Invalid reservation information raises `ValueError`; unavailable reserved nights raise `UnavailableError`.

**Pricing:** $200 per night + a one-time $50 cleaning fee + $25 per night for **each guest above five**.

For valid bookings, the expected total is:

`total = (nights × 200) + 50 + (max(guests − 5, 0) × nights × 25)`

All dates below use ISO format `YYYY-MM-DD`. Unless otherwise stated, test cases use **3 guests**, **2026-11-10 check-in**, **2026-11-13 checkout** (3 nights), and full availability. Each case changes only the specified classification. A standard 3-night, 3-guest booking costs **$650**.

# 2. Equivalence partition tables

# A. Number of nights

| Partition | Range or condition | Representative value | Expected result |
|---|---|---|---|
| Invalid low | 0 or fewer nights | 0 nights | `ValueError` |
| Valid | 1–31 nights inclusive | 1, 3, or 31 nights | Accepted if all other conditions are valid; price uses the actual nights |
| Invalid high | More than 31 nights | 32 nights | `ValueError` |

# B. Number of guests

| Partition | Range or condition | Representative value | Expected result |
|---|---|---|---|
| Invalid low | Fewer than 1 guest | 0 guests | `ValueError` |
| Valid, standard rate | 1–5 guests | 3 guests | Accepted; no extra guest fee |
| Valid, additional fee | 6–12 guests | 6 or 12 guests | Accepted; $25 per extra guest per night |
| Invalid high | More than 12 guests | 13 guests | `ValueError` |

# C. Check-in and checkout dates

| Partition | Range or condition | Representative value | Expected result |
|---|---|---|---|
| Invalid, same date | Check-in equals checkout | Nov 10 → Nov 10 | `ValueError` (0 nights) |
| Invalid, reversed dates | Check-in after checkout | Nov 13 → Nov 10 | `ValueError` |
| Valid, ordered dates | Check-in before checkout and stay is 1–31 nights | Nov 10 → Nov 13 | Accepted if available; 3 nights, excluding checkout |
| Invalid, excessive span | Ordered dates but stay exceeds 31 nights | Nov 10 → Dec 12 | `ValueError` (32 nights) |

The date-span cases overlap the nights classification intentionally: they verify the date-to-night calculation and the maximum stay rule together. The requirements do not define behavior for malformed date strings, so those are not assigned an invented expected exception here.

# D. Availability

| Partition | Range or condition | Representative value | Expected result |
|---|---|---|---|
| Valid, all nights available | Every reserved night is available | Nov 10, 11, 12 available | Accepted |
| Invalid, first night unavailable | Check-in night is unavailable | Nov 10 unavailable | `UnavailableError` |
| Invalid, middle night unavailable | At least one interior night is unavailable | Nov 11 unavailable | `UnavailableError` |
| Invalid, last reserved night unavailable | Night before checkout unavailable | Nov 12 unavailable | `UnavailableError` |
| Valid, checkout date unavailable | All reserved nights available; checkout date unavailable | Nov 13 unavailable only | Accepted; checkout is excluded |

# 3. Test cases

**Common defaults:** 3 guests, check-in `2026-11-10`, checkout `2026-11-13`, and all reserved nights available. Values shown in a test case replace these defaults.

| Test ID | Classification | Partition | Input | Expected result |
|---|---|---|---|---|
| TC01 | Nights | Invalid low | Nov 10 → Nov 10; 0 nights | `ValueError` |
| TC02 | Nights | Valid, minimum | Nov 10 → Nov 11; 1 night | Accepted; **$250** |
| TC03 | Nights | Valid, maximum | Nov 10 → Dec 11; 31 nights | Accepted; **$6,250** |
| TC04 | Nights | Invalid high | Nov 10 → Dec 12; 32 nights | `ValueError` |
| TC05 | Guests | Invalid low | 0 guests | `ValueError` |
| TC06 | Guests | Valid, standard rate | 3 guests; 3 nights | Accepted; **$650**, no extra guest fee |
| TC07 | Guests | Valid, additional fee | 6 guests; 3 nights | Accepted; **$725** ($75 extra guest fee) |
| TC08 | Guests | Valid, additional fee upper limit | 12 guests; 3 nights | Accepted; **$1,175** ($525 extra guest fee) |
| TC09 | Guests | Invalid high | 13 guests | `ValueError` |
| TC10 | Dates | Invalid, same date | Nov 10 → Nov 10 | `ValueError` |
| TC11 | Dates | Invalid, reversed dates | Nov 13 → Nov 10 | `ValueError` |
| TC12 | Dates | Valid, ordered dates | Nov 10 → Nov 13 | Accepted; 3 reserved nights; **$650** |
| TC13 | Dates | Invalid, excessive span | Nov 10 → Dec 12 | `ValueError` |
| TC14 | Availability | Valid, all nights available | Nov 10, 11, 12 available | Accepted; **$650**|
| TC15 | Availability | Invalid, first night unavailable | Nov 10 unavailable; Nov 11–12 available | `UnavailableError` |
| TC16 | Availability | Invalid, middle night unavailable | Nov 11 unavailable; Nov 10 and 12 available | `UnavailableError` |
| TC17 | Availability | Invalid, last reserved night unavailable | Nov 12 unavailable; Nov 10–11 available | `UnavailableError` |
| TC18 | Availability | Valid, checkout date unavailable | Nov 10–12 available; Nov 13 unavailable | Accepted; **$650** (checkout is not reserved) |

# 4. Why equivalence partitioning helps

Equivalence partitioning groups inputs expected to behave alike. Instead of testing every possible guest count, date, and stay length, I choose representative values from valid and invalid groups. This reduces repeated tests while still checking important rules, including limits, extra guest charges, invalid dates, and unavailable nights. I included some boundary values to make the tests clearer.

# 5. Test design status

These are **planned test cases**, not executed results. The application source code was not provided, so actual pass/fail outcomes cannot be reported. The expected results are based on the project requirements.

# 6. AI transparency

ChatGPT assisted with organizing the equivalence partitions, calculating expected prices, and drafting the test cases and README. I should review and confirm the work before submitting it.

# 7. Reference

GeeksforGeeks. (2026, July 3). *Equivalence partitioning method*. https://www.geeksforgeeks.org/software-engineering/equivalence-partitioning-method/
