-- ============================================================
-- Star Schema: Airline Passenger Satisfaction
-- ============================================================

-- DIMENSION: Passenger
CREATE TABLE dim_passenger (
    passenger_key   INT PRIMARY KEY,
    gender          VARCHAR(10),
    customer_type   VARCHAR(20),
    age_band        VARCHAR(15)
);

-- DIMENSION: Flight
CREATE TABLE dim_flight (
    flight_key      INT PRIMARY KEY,
    type_of_travel  VARCHAR(20),
    class           VARCHAR(20)
);

-- DIMENSION: Satisfaction (outcome)
CREATE TABLE dim_satisfaction (
    satisfaction_key INT PRIMARY KEY,
    satisfaction     VARCHAR(30)
);

-- DIMENSION: Distance
CREATE TABLE dim_distance (
    distance_key    INT PRIMARY KEY,
    distance_band   VARCHAR(30)
);

-- FACT TABLE
CREATE TABLE fact_flight_satisfaction (
    flight_survey_id            INT PRIMARY KEY,
    passenger_key               INT REFERENCES dim_passenger(passenger_key),
    flight_key                  INT REFERENCES dim_flight(flight_key),
    satisfaction_key            INT REFERENCES dim_satisfaction(satisfaction_key),
    distance_key                INT REFERENCES dim_distance(distance_key),

    -- Degenerate / numeric measures
    age                         INT,
    flight_distance             INT,
    departure_delay_in_minutes  INT,
    arrival_delay_in_minutes    INT,

    -- Service rating measures (1-5 scale)
    inflight_wifi_service               TINYINT,
    departure_arrival_time_convenient   TINYINT,
    ease_of_online_booking              TINYINT,
    gate_location                       TINYINT,
    food_and_drink                      TINYINT,
    online_boarding                     TINYINT,
    seat_comfort                        TINYINT,
    inflight_entertainment              TINYINT,
    on_board_service                    TINYINT,
    leg_room_service                    TINYINT,
    baggage_handling                    TINYINT,
    checkin_service                     TINYINT,
    inflight_service                    TINYINT,
    cleanliness                         TINYINT
);

-- ============================================================
-- Example analytical query: satisfaction rate by class & travel type
-- ============================================================
SELECT
    f.type_of_travel,
    f.class,
    COUNT(*) AS total_flights,
    SUM(CASE WHEN s.satisfaction = 'satisfied' THEN 1 ELSE 0 END) AS satisfied_count,
    ROUND(100.0 * SUM(CASE WHEN s.satisfaction = 'satisfied' THEN 1 ELSE 0 END) / COUNT(*), 1) AS satisfaction_pct,
    AVG(fact.arrival_delay_in_minutes) AS avg_arrival_delay
FROM fact_flight_satisfaction fact
JOIN dim_flight f         ON fact.flight_key = f.flight_key
JOIN dim_satisfaction s   ON fact.satisfaction_key = s.satisfaction_key
GROUP BY f.type_of_travel, f.class
ORDER BY satisfaction_pct DESC;
