Learning Practice: Black-Box Testing, Part 1
Application: North Vacation Booking
Testing method: Black-box testing — equivalence partitioning
Assignment: Learning Practice: Black-Box Testing, Part 1
Deliverable: Equivalence partition tables and planned test cases (no coding required)
1. Project overview
North Vacation Booking is a vacation rental application. My goal is to design test cases from its requirements without seeing or running its code.  A valid reservation has 1–31 nights, 1–12 guests, check-in before checkout, and every reserved night available. Checkout is not a reserved night. Invalid reservation information raises ValueError; unavailable reserved nights raise UnavailableError.
Pricing: $200 per night + a one-time $50 cleaning fee + $25 per night for each guest above five.
For valid bookings, the expected total is:
total = (nights × 200) + 50 + (max(guests − 5, 0) × nights × 25)
All dates below use ISO format YYYY-MM-DD. Unless otherwise stated, test cases use 3 guests, 2026-11-10 check-in, 2026-11-13 checkout (3 nights), and full availability. Each case changes only the specified classification. A standard 3-night, 3-guest booking costs $650.
2. How I selected the partitions
I identified four areas to test: the length of the stay, number of guests, check-in and checkout dates, and night-by-night availability. I divided each area into valid and invalid conditions, then chose representative inputs for each one. For valid bookings, I also calculated the expected price.
3. Equivalence partition tables
A. Number of nights
Partition	Range or condition	Representative value	Expected result
Invalid low	0 or fewer nights	0 nights	ValueError
Valid	1–31 nights inclusive	1, 3, or 31 nights	Accepted if all other conditions are valid; price uses the actual nights
Invalid high	More than 31 nights	32 nights	ValueError


B. Number of guests
Partition	Range or condition	Representative value	Expected result
Invalid low	Fewer than 1 guest	0 guests	ValueError
Valid, standard rate	1–5 guests	3 guests	Accepted; no extra guest fee
Valid, additional fee	6–12 guests	6 or 12 guests	Accepted; $25 per extra guest per night
Invalid high	More than 12 guests	13 guests	ValueError


C. Check-in and checkout dates
Partition	Range or condition	Representative value	Expected result
Invalid, same date	Check-in equals checkout	Nov 10 → Nov 10	ValueError (0 nights)
Invalid, reversed dates	Check-in after checkout	Nov 13 → Nov 10	ValueError
Valid, ordered dates	Check-in before checkout and stay is 1–31 nights	Nov 10 → Nov 13	Accepted if available; 3 nights, excluding checkout
Invalid, excessive span	Ordered dates but stay exceeds 31 nights	Nov 10 → Dec 12	ValueError (32 nights)


Some date cases also check the number-of-nights rules: they verify the date-to-night calculation and the maximum stay rule together. The requirements do not explain how malformed date strings are handled, so I have not assumed an error type for those inputs.
D. Availability
Partition	Range or condition	Representative value	Expected result
Valid, all nights available	Every reserved night is available	Nov 10, 11, 12 available	Accepted
Invalid, first night unavailable	Check-in night is unavailable	Nov 10 unavailable	UnavailableError
Invalid, middle night unavailable	At least one interior night is unavailable	Nov 11 unavailable	UnavailableError
Invalid, last reserved night unavailable	Night before checkout unavailable	Nov 12 unavailable	UnavailableError
Valid, checkout date unavailable	All reserved nights available; checkout date unavailable	Nov 13 unavailable only	Accepted; checkout is excluded


4. Test cases
Common defaults: 3 guests, check-in 2026-11-10, checkout 2026-11-13, and all reserved nights available (including longer stays when needed). Values shown in a test case replace these defaults.
Test ID	Classification	Partition	Input	Expected result
TC01	Nights	Invalid low	Nov 10 → Nov 10; 0 nights	ValueError
TC02	Nights	Valid, minimum	Nov 10 → Nov 11; 1 night	Accepted; $250
TC03	Nights	Valid, maximum	Nov 10 → Dec 11; 31 nights	Accepted; $6,250
TC04	Nights	Invalid high	Nov 10 → Dec 12; 32 nights	ValueError
TC05	Guests	Invalid low	0 guests	ValueError
TC06	Guests	Valid, standard rate	3 guests; 3 nights	Accepted; $650, no extra guest fee
TC07	Guests	Valid, additional fee	6 guests; 3 nights	Accepted; $725 ($75 extra guest fee)
TC08	Guests	Valid, additional fee upper limit	12 guests; 3 nights	Accepted; $1,175 ($525 extra guest fee)
TC09	Guests	Invalid high	13 guests	ValueError
TC10	Dates	Invalid, same date	Nov 10 → Nov 10	ValueError
TC11	Dates	Invalid, reversed dates	Nov 13 → Nov 10	ValueError
TC12	Dates	Valid, ordered dates	Nov 10 → Nov 13	Accepted; 3 reserved nights; $650
TC13	Dates	Invalid, excessive span	Nov 10 → Dec 12	ValueError
TC14	Availability	Valid, all nights available	Nov 10, 11, 12 available	Accepted; $650
TC15	Availability	Invalid, first night unavailable	Nov 10 unavailable; Nov 11–12 available	UnavailableError
TC16	Availability	Invalid, middle night unavailable	Nov 11 unavailable; Nov 10 and 12 available	UnavailableError
TC17	Availability	Invalid, last reserved night unavailable	Nov 12 unavailable; Nov 10–11 available	UnavailableError
TC18	Availability	Valid, checkout date unavailable	Nov 10–12 available; Nov 13 unavailable	Accepted; $650 (checkout is not reserved)


5. Why equivalence partitioning helps
Equivalence partitioning groups inputs expected to behave alike. Instead of testing every possible guest count, date, and stay length, I choose representative values from valid and invalid groups. This reduces repeated tests while still checking important rules, including limits, extra guest charges, invalid dates, and unavailable nights. I included some boundary values to make the tests clearer.
6. Test design status
These are planned test cases, not executed results. The application source code was not provided, so actual pass/fail outcomes cannot be reported. The expected results are based on the project requirements.
7. AI transparency
I used ChatGPT to help organize the tables, draft test cases, check the price calculations, and improve the wording of this README. The expected results are based on the assignment requirements, not on running the application.
8. Reference
GeeksforGeeks. (2026, July 3). Equivalence partitioning method. https://www.geeksforgeeks.org/software-engineering/equivalence-partitioning-method/
