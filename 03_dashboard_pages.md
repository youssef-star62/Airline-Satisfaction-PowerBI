// ============================================================
// DAX MEASURES — Airline Passenger Satisfaction
// Drop these into a "Measures" table in Power BI.
// Table refs assume: fact_flight_satisfaction, dim_passenger,
//                    dim_flight, dim_satisfaction, dim_distance
// ============================================================


// ============================================================
// 1. SATISFACTION KPIs (headline metrics)
// ============================================================

Total Surveys =
COUNTROWS ( fact_flight_satisfaction )

Satisfied Count =
CALCULATE (
    [Total Surveys],
    dim_satisfaction[satisfaction] = "satisfied"
)

Dissatisfied Count =
CALCULATE (
    [Total Surveys],
    dim_satisfaction[satisfaction] = "neutral or dissatisfied"
)

Satisfaction Rate % =
DIVIDE ( [Satisfied Count], [Total Surveys], 0 )

// Format as percentage in the model. Industry benchmark ~45%.

Satisfaction Rate Target % = 0.50

Satisfaction Gap vs Target =
[Satisfaction Rate %] - [Satisfaction Rate Target %]


// ============================================================
// 2. SERVICE QUALITY KPIs (the 14 rating columns)
// ============================================================

// --- Average ratings per service (1-5 scale) ---
Avg Wifi Service              = AVERAGE ( fact_flight_satisfaction[inflight_wifi_service] )
Avg Time Convenience          = AVERAGE ( fact_flight_satisfaction[departure_arrival_time_convenient] )
Avg Online Booking            = AVERAGE ( fact_flight_satisfaction[ease_of_online_booking] )
Avg Gate Location             = AVERAGE ( fact_flight_satisfaction[gate_location] )
Avg Food and Drink            = AVERAGE ( fact_flight_satisfaction[food_and_drink] )
Avg Online Boarding           = AVERAGE ( fact_flight_satisfaction[online_boarding] )
Avg Seat Comfort              = AVERAGE ( fact_flight_satisfaction[seat_comfort] )
Avg Inflight Entertainment    = AVERAGE ( fact_flight_satisfaction[inflight_entertainment] )
Avg On-board Service          = AVERAGE ( fact_flight_satisfaction[on-board_service] )
Avg Leg Room                  = AVERAGE ( fact_flight_satisfaction[leg_room_service] )
Avg Baggage Handling          = AVERAGE ( fact_flight_satisfaction[baggage_handling] )
Avg Checkin Service           = AVERAGE ( fact_flight_satisfaction[checkin_service] )
Avg Inflight Service          = AVERAGE ( fact_flight_satisfaction[inflight_service] )
Avg Cleanliness               = AVERAGE ( fact_flight_satisfaction[cleanliness] )

// --- Overall service score (average across all 14 ratings) ---
Overall Service Score =
VAR Services =
    [Avg Wifi Service] + [Avg Time Convenience] + [Avg Online Booking]
    + [Avg Gate Location] + [Avg Food and Drink] + [Avg Online Boarding]
    + [Avg Seat Comfort] + [Avg Inflight Entertainment]
    + [Avg On-board Service] + [Avg Leg Room] + [Avg Baggage Handling]
    + [Avg Checkin Service] + [Avg Inflight Service] + [Avg Cleanliness]
RETURN
    DIVIDE ( Services, 14 )

// --- % Top Box: ratings of 4 or 5 (e.g. seat comfort) ---
% Top Box Seat Comfort =
DIVIDE (
    CALCULATE ( [Total Surveys], fact_flight_satisfaction[seat_comfort] >= 4 ),
    [Total Surveys],
    0
)

% Top Box Wifi =
DIVIDE (
    CALCULATE ( [Total Surveys], fact_flight_satisfaction[inflight_wifi_service] >= 4 ),
    [Total Surveys],
    0
)

% Top Box Cleanliness =
DIVIDE (
    CALCULATE ( [Total Surveys], fact_flight_satisfaction[cleanliness] >= 4 ),
    [Total Surveys],
    0
)

// --- % Bottom Box: ratings of 1 or 2 — your pain points ---
% Bottom Box Wifi =
DIVIDE (
    CALCULATE ( [Total Surveys], fact_flight_satisfaction[inflight_wifi_service] <= 2 ),
    [Total Surveys],
    0
)

% Bottom Box Food =
DIVIDE (
    CALCULATE ( [Total Surveys], fact_flight_satisfaction[food_and_drink] <= 2 ),
    [Total Surveys],
    0
)


// ============================================================
// 3. OPERATIONAL KPIs (delays)
// ============================================================

Avg Departure Delay (min) =
AVERAGE ( fact_flight_satisfaction[departure_delay_in_minutes] )

Avg Arrival Delay (min) =
AVERAGE ( fact_flight_satisfaction[arrival_delay_in_minutes] )

Max Arrival Delay (min) =
MAX ( fact_flight_satisfaction[arrival_delay_in_minutes] )

// On-time = arrived with 0 minutes delay
On-Time Arrival % =
DIVIDE (
    CALCULATE (
        [Total Surveys],
        fact_flight_satisfaction[arrival_delay_in_minutes] = 0
    ),
    [Total Surveys],
    0
)

// Severely delayed = arrival delay > 15 minutes (industry threshold)
% Severely Delayed =
DIVIDE (
    CALCULATE (
        [Total Surveys],
        fact_flight_satisfaction[arrival_delay_in_minutes] > 15
    ),
    [Total Surveys],
    0
)

Avg Flight Distance =
AVERAGE ( fact_flight_satisfaction[flight_distance] )


// ============================================================
// 4. SEGMENT KPIs (slicing by passenger/flight)
// ============================================================

// Satisfaction rate among loyal customers only
Loyal Customer Satisfaction % =
CALCULATE (
    [Satisfaction Rate %],
    dim_passenger[customer_type] = "Loyal Customer"
)

// Satisfaction rate among business class only
Business Class Satisfaction % =
CALCULATE (
    [Satisfaction Rate %],
    dim_flight[class] = "Business"
)

// Satisfaction rate among business travelers only
Business Travel Satisfaction % =
CALCULATE (
    [Satisfaction Rate %],
    dim_flight[type_of_travel] = "Business travel"
)

// Loyalty share — what % of surveys are from loyal customers
Loyal Customer Share % =
DIVIDE (
    CALCULATE (
        [Total Surveys],
        dim_passenger[customer_type] = "Loyal Customer"
    ),
    [Total Surveys],
    0
)


// ============================================================
// 5. BONUS: Conditional formatting / KPI card helpers
// ============================================================

// Color-code the satisfaction rate card (use in conditional formatting)
Satisfaction RAG Color =
SWITCH (
    TRUE (),
    [Satisfaction Rate %] >= 0.55, "#1D9E75",  // green
    [Satisfaction Rate %] >= 0.45, "#EF9F27",  // amber
    "#E24B4A"                                  // red
)

// Top driver of dissatisfaction — lowest avg rating service name
Lowest Rated Service =
VAR Ratings =
    DATATABLE (
        "Service", STRING,
        "Score",   DOUBLE,
        {
            { "Wifi",            BLANK () },
            { "Online Booking",  BLANK () },
            { "Food",            BLANK () },
            { "Seat Comfort",    BLANK () },
            { "Entertainment",   BLANK () },
            { "Cleanliness",     BLANK () }
        }
    )
// In practice use a calculated table or just reference the lowest measure manually.
// The DATATABLE pattern above is a placeholder — real implementation uses MINX
// over an unpivoted "Service Ratings" table (preferred star-schema pattern).
RETURN BLANK ()
