# Crop-management

                               EST Practical Activity Report  
                                           Submitted for  
 
           DATABASE MANAGEMENT SYSTEM (UCS310) 

                                                     
Crop Management System 
                         Computer Science and Engineering Department 
                                                   TIET, Patiala 
                      Jan-May 2023 
  
 
  
INDEX 
Sr. No.  
1. 
Title Page 
Page No. 
1 
2. 
Index 
2 
3. 
Introduction 
3 
4. 
ER-Diagram 
5 
5. 
ER to Table 
6 
6. 
Normalization 
7 
7. 
PL/SQL 
8 
8. 
Conclusion 
35 
INTRODUCTION 
The Crop Management System is an essential tool for modern-day farmers, as it streamlines 
various aspects of crop production, management, and marketing. The system aims to provide 
a comprehensive platform that enables farmers to efficiently manage their crops from planting 
to harvest, using data-driven insights to maximize yields and optimize profits. 
The Crop Management System developed in this project report is an innovative approach to 
managing crop operations and improving productivity. The report covers the design and 
implementation of the system using PL/SQL programming language, with a focus on using a 
relational database with multiple tables to store data related to crops, production, buyers, 
weather, owners, fertilizers, and market information. To manage the data efficiently, PL/SQL 
procedures have been developed to allow for the easy insertion, updating, and deletion of 
records from each of these tables. 
The report also discusses the use of cursors to retrieve data from the tables as well as the 
development of triggers to automate certain actions. Overall, this project report provides a 
comprehensive guide to building a crop management system using PL/SQL, and should serve 
as a useful resource for those looking to implement similar systems in the future. 
This project aims to develop a CMS using DBMS and SQL to streamline crop management 
operations and improve productivity. 
Requirements Analysis: 
1. User Management: The system should allow the administrator to manage user accounts, 
create new users, and set user permissions. 
2. Crop Management: The system should allow the user to manage crop-related 
information, such as crop details, crop category, and crop quantity. 
3. Production Management: The system should allow the user to manage production
related information, such as production date, production quantity, and crop ID. 
4. Fertilizer Management: The system should allow the user to manage fertilizer-related 
information, such as fertilizer details, fertilizer quantity, and fertilizer usage. 
5. Market Information: The system should provide the user with relevant market 
information, such as market prices and demand for different crops. 
6. Weather Information: The system should provide the user with relevant weather 
information, such as temperature and rainfall data. 
7. Reporting: The system should provide the user with various reports, such as crop 
production reports, fertilizer usage reports, and market demand reports. 
8. Data Security: The system should have robust data security features to prevent 
unauthorized access and ensure data confidentiality. 
9. Scalability: The system should be scalable to accommodate an increasing amount of 
data as the crop management system grows. 
ER DIAGRAM 
ER to Tables 
PL/SQL 
CREATION OF TABLES 
CREATE TABLE Seed( 
Seed_ID NUMBER(7) NOT NULL, 
SeedName VARCHAR(15), 
PRIMARY KEY (Seed_ID) 
); 
CREATE TABLE Buyer(  
Buyer_ID NUMBER(7) NOT NULL, 
BuyerName VARCHAR(50), 
BuyerPhone NUMBER(15) NOT NULL, 
Price NUMBER(15) NOT NULL, 
PRIMARY KEY (Buyer_ID) 
); 
CREATE TABLE Weather ( 
WCondition VARCHAR(10), 
Humidity NUMBER(7,2) NOT NULL, 
Temperature NUMBER(7,2) NOT NULL, 
Rainfall NUMBER(7,2) NOT NULL, 
PRIMARY KEY (Temperature) 
); 
CREATE TABLE Owner ( 
Owner_ID NUMBER(7) not null, 
OwnerName VARCHAR(50) unique, 
OwnerAddress VARCHAR(50), 
OwnerPhone NUMBER(15) NOT NULL, 
PRIMARY KEY (Owner_ID) 
); 
CREATE TABLE Fertilizer ( 
FertilizerName VARCHAR(50), 
FertilizerID NUMBER(7) NOT NULL, 
PRIMARY KEY (FertilizerName) 
); 
CREATE TABLE Production ( 
IN_Stock NUMBER(5) UNIQUE, 
Owner_ID NUMBER(7) NOT NULL, 
BUYER_ID NUMBER(7) NOT NULL, 
Produced NUMBER(15) UNIQUE, 
Seed_ID NUMBER(7) NOT NULL, 
FertilizerName VARCHAR(50), 
Temperature Number(7,2), 
PRIMARY KEY (IN_Stock, Produced), 
FOREIGN KEY (Buyer_ID) REFERENCES Buyer(Buyer_ID) ON DELETE CASCADE, 
FOREIGN KEY (Owner_ID) REFERENCES Owner(Owner_ID) ON DELETE CASCADE, 
FOREIGN KEY (Seed_ID) REFERENCES Seed(Seed_ID) ON DELETE CASCADE, 
FOREIGN KEY (FertilizerName) REFERENCES Fertilizer(FertilizerName) ON DELETE 
CASCADE, 
FOREIGN KEY (Temperature) REFERENCES Weather(Temperature) ON DELETE CASCADE 
); 
CREATE TABLE Crop ( 
Crop_ID NUMBER(7) NOT NULL, 
CropCategory VARCHAR(50), 
Produced NUMBER(15) unique, 
FOREIGN KEY (Produced) REFERENCES Production(Produced) ON DELETE CASCADE 
); 
CREATE TABLE MarketInfo ( 
In_Stock NUMBER(5) UNIQUE, 
Price NUMBER(15) NOT NULL, 
Buyer_ID NUMBER(7) NOT NULL, 
MarketAddress VARCHAR(50), 
PRIMARY KEY (Price), 
FOREIGN KEY (IN_Stock) REFERENCES Production(IN_Stock) ON DELETE CASCADE, 
FOREIGN KEY (Buyer_ID) REFERENCES Buyer(Buyer_ID) ON DELETE CASCADE 
); 
FUNCTIONS: 
// a function total_production that takes crop_id as input and returns the total production quantity for that 
crop 
CREATE OR REPLACE FUNCTION total_production(crop_id NUMBER) 
RETURN NUMBER 
IS 
total_prod NUMBER(15) := 0; 
BEGIN 
SELECT SUM(Produced) INTO total_prod 
FROM Production 
WHERE Crop_ID = crop_id; 
RETURN total_prod; 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
RETURN NULL; 
END; 
//change function take market_address as input and return in_stock, price, buyer_id 
CREATE OR REPLACE FUNCTION market_information(market_address VARCHAR) 
RETURN SYS_REFCURSOR 
IS 
market_data SYS_REFCURSOR; 
BEGIN 
OPEN market_data FOR 
SELECT IN_Stock, Price, Buyer_ID  
FROM MarketInfo 
WHERE MarketAddress = market_address; 
RETURN market_data; 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
RETURN NULL; 
END; 
//A function that calculates the total amount of fertilizer used by an owner: 
CREATE OR REPLACE FUNCTION get_total_fertilizer_used(owner_id NUMBER)  
RETURN NUMBER  
IS  
total_fertilizer NUMBER;  
BEGIN  
SELECT COUNT(*) INTO total_fertilizer  
FROM production  
WHERE owner_id = owner_id;  
RETURN total_fertilizer;  
END;  
/ 
A function that returns the number of owners who have produced a particular crop category 
CREATE OR REPLACE FUNCTION get_num_owners(crop_category VARCHAR)  
RETURN NUMBER  
IS  
num_owners NUMBER;  
BEGIN  
SELECT COUNT(DISTINCT owner_id) INTO num_owners  
FROM production p, crop c, seed s  
WHERE p.seed_id = s.seed_id AND c.produced = p.produced  
AND c.cropcategory = crop_category;  
RETURN num_owners;  
END;  
/ 
//Determine the average market price for a specific crop based on historical data: 
CREATE OR REPLACE FUNCTION average_crop_price(crop_name IN VARCHAR2) RETURN 
NUMBER 
IS 
avg_price NUMBER := 0; 
BEGIN 
SELECT AVG(Price) 
INTO avg_price 
FROM MarketInfo 
WHERE IN_Stock = (SELECT IN_Stock FROM Production WHERE Seed_ID = (SELECT Seed_ID 
FROM Seed WHERE SeedName = crop_name)); 
RETURN avg_price; 
END; 
// a function total_production that takes crop_id as input and returns the total production quantity for 
that crop 
CREATE OR REPLACE FUNCTION total_production(crop_id NUMBER) 
RETURN NUMBER 
IS 
total_prod NUMBER(15) := 0; 
BEGIN 
SELECT SUM(Produced) INTO total_prod 
FROM Production 
WHERE Crop_ID = crop_id; 
RETURN total_prod; 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
RETURN NULL; 
END; 
//calling function 
SELECT total_production(30001) as Total_Production FROM dual; 
//change function take market_address as input and return in_stock, price, buyer_id 
CREATE OR REPLACE FUNCTION market_information(market_address VARCHAR) 
RETURN SYS_REFCURSOR 
IS 
market_data SYS_REFCURSOR; 
BEGIN 
OPEN market_data FOR 
SELECT IN_Stock, Price, Buyer_ID  
FROM MarketInfo 
WHERE MarketAddress = market_address; 
RETURN market_data; 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
RETURN NULL; 
END; 
// Calling fucnction 
select market_information('Dhaka') as market_info from dual; 
//A function that calculates the total amount of fertilizer used by an owner: 
CREATE OR REPLACE FUNCTION get_total_fertilizer_used(owner_id NUMBER)  
RETURN NUMBER  
IS  
total_fertilizer NUMBER;  
BEGIN  
SELECT COUNT(*) INTO total_fertilizer  
FROM production  
WHERE owner_id = owner_id;  
RETURN total_fertilizer;  
END;  
/ 
//Calling Function 
select get_total_fertilizer_used(1000) as used_fertilizer from dual; 
PROCEDURES: 
CREATE OR REPLACE PROCEDURE insert_seed( 
p_Seed_ID IN Seed.Seed_ID%TYPE, 
p_SeedName IN Seed.SeedName%TYPE 
) AS 
BEGIN 
INSERT INTO Seed (Seed_ID, SeedName) VALUES (p_Seed_ID, p_SeedName); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Seed inserted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLCODE || ' - ' || SQLERRM); 
ROLLBACK; 
END insert_seed; 
CREATE OR REPLACE PROCEDURE insert_buyer( 
p_Buyer_ID IN Buyer.Buyer_ID%TYPE, 
p_BuyerName IN Buyer.BuyerName%TYPE, 
p_BuyerPhone IN Buyer.BuyerPhone%TYPE, 
p_Price IN Buyer.Price%TYPE 
) AS 
BEGIN 
INSERT INTO Buyer (Buyer_ID, BuyerName, BuyerPhone, Price) VALUES (p_Buyer_ID, 
p_BuyerName, p_BuyerPhone, p_Price); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Buyer inserted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLCODE || ' - ' || SQLERRM); 
ROLLBACK; 
END insert_buyer; 
CREATE OR REPLACE PROCEDURE insert_weather( 
p_WCondition IN Weather.WCondition%TYPE, 
p_Humidity IN Weather.Humidity%TYPE, 
p_Temperature IN Weather.Temperature%TYPE, 
p_Rainfall IN Weather.Rainfall%TYPE 
) AS 
BEGIN 
INSERT INTO Weather (WCondition, Humidity, Temperature, Rainfall) VALUES (p_WCondition, 
p_Humidity, p_Temperature, p_Rainfall); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Weather inserted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLCODE || ' - ' || SQLERRM); 
ROLLBACK; 
END insert_weather; 
CREATE OR REPLACE PROCEDURE insert_owner( 
p_Owner_ID IN Owner.Owner_ID%TYPE, 
p_OwnerName IN Owner.OwnerName%TYPE, 
p_OwnerAddress IN Owner.OwnerAddress%TYPE, 
p_OwnerPhone IN Owner.OwnerPhone%TYPE 
) AS 
BEGIN 
INSERT INTO Owner (Owner_ID, OwnerName, OwnerAddress, OwnerPhone) VALUES 
(p_Owner_ID, p_OwnerName, p_OwnerAddress, p_OwnerPhone); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Owner inserted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLCODE || ' - ' || SQLERRM); 
ROLLBACK; 
END insert_owner; 
CREATE OR REPLACE PROCEDURE insert_fertilizer( 
p_fertilizername IN Fertilizer.FertilizerName%TYPE, 
p_fertilizerid IN Fertilizer.FertilizerID%TYPE 
) 
IS 
BEGIN 
INSERT INTO fertilizer(fertilizername, fertilizerid) 
VALUES(p_fertilizername, p_fertilizerid); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Fertilizer ' || p_fertilizername || ' added successfully.'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END insert_fertilizer; 
CREATE OR REPLACE PROCEDURE insert_production( 
p_in_stock IN Production.IN_Stock%TYPE, 
p_owner_id IN Production.Owner_ID%TYPE, 
p_buyer_id IN Production.BUYER_ID%TYPE, 
p_produced IN Production.Produced%TYPE, 
p_seed_id IN Production.Seed_ID%TYPE, 
p_fertilizername IN Production.FertilizerName%TYPE, 
p_temperature IN Production.Temperature%TYPE 
) 
IS 
BEGIN 
INSERT INTO production(in_stock, owner_id, buyer_id, produced, seed_id, fertilizername, 
temperature) 
VALUES(p_in_stock, p_owner_id, p_buyer_id, p_produced, p_seed_id, p_fertilizername, 
p_temperature); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Production record added successfully.'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END insert_production; 
CREATE OR REPLACE PROCEDURE insert_crop( 
v_crop_id crop.crop_id%TYPE, 
v_crop_category crop.cropcategory%TYPE, 
v_produced crop.produced%TYPE 
) 
AS 
BEGIN 
INSERT INTO crop (crop_id, cropcategory, produced) 
VALUES (v_crop_id, v_crop_category, v_produced); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Values inserted successfully into the crop table.'); 
EXCEPTION 
WHEN OTHERS THEN 
ROLLBACK; 
DBMS_OUTPUT.PUT_LINE('Error inserting values into the crop table: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE insert_marketinfo( 
v_in_stock marketinfo.in_stock%TYPE, 
v_price marketinfo.price%TYPE, 
v_buyer_id marketinfo.buyer_id%TYPE, 
v_marketaddress marketinfo.marketAddress%TYPE 
) 
AS 
BEGIN 
INSERT INTO marketinfo (in_stock, price, buyer_id, marketAddress) 
VALUES (v_in_stock, v_price, v_buyer_id, v_marketaddress); 
COMMIT; 
DBMS_OUTPUT.PUT_LINE('Values inserted successfully into the marketinfo table.'); 
EXCEPTION 
WHEN OTHERS THEN 
ROLLBACK; 
DBMS_OUTPUT.PUT_LINE('Error inserting values into the marketinfo table: ' || SQLERRM); 
END; 
----------------------------------------------------------------------- 
begin 
insert_marketinfo (18,4371,10001,'Dhaka' ); 
insert_marketinfo (14,3275,10002,'Kishoregonj' ); 
insert_marketinfo (11,4349,10003,'Mymensingh' ); 
insert_marketinfo (15,4648,10004,'Tangail' ); 
insert_marketinfo (17,4253,10005,'Faridpur' ); 
insert_marketinfo (13,3669,10006,'Noakhali' ); 
insert_marketinfo (19,4256,10007,'Comilla' ); 
insert_marketinfo (20,3205,10008,'Bogra' ); 
insert_marketinfo (16,4807,10009,'Pabna' ); 
insert_marketinfo (10,3816,10010,'Jessore' ); 
end; 
/ 
select * from marketinfo; 
begin 
insert_crop (30001, 'Cereal Grain', 424); 
insert_crop (30002, 'Legume', 356); 
insert_crop (30003, 'Pericap', 364); 
insert_crop (30004, 'Legume', 425); 
insert_crop (30005, 'Poaceae', 402); 
insert_crop (30006, 'Cereal Grain', 377); 
insert_crop (30007, 'Fruit', 372); 
insert_crop (30008, 'Fruit', 365); 
insert_crop (30009, 'Legume', 451); 
insert_crop (30010, 'Boro', 445); 
end; 
/ 
select * from crop; 
SHOW PROCEDURE: 
CREATE OR REPLACE PROCEDURE show_buyer_records AS 
BEGIN 
FOR buyer_rec IN (SELECT * FROM Buyer)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Buyer ID: ' || buyer_rec.Buyer_ID || ', Buyer Name: ' || 
buyer_rec.BuyerName ||  
', Buyer Phone: ' || buyer_rec.BuyerPhone || ', Price: ' || buyer_rec.Price); 
END LOOP; 
END; 
/ 
begin 
show_buyer_records; 
end; 
/ 

//Updation Procedures 
CREATE OR REPLACE PROCEDURE update_seed( 
p_Seed_ID Seed.Seed_ID%TYPE, 
p_SeedName Seed.SeedName%TYPE 
) 
IS 
BEGIN 
UPDATE Seed 
SET SeedName = p_SeedName 
WHERE Seed_ID = p_Seed_ID; 
DBMS_OUTPUT.PUT_LINE('Seed record updated successfully.'); 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
DBMS_OUTPUT.PUT_LINE('Seed with ID ' || p_Seed_ID || ' does not exist.'); 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error updating Seed record: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE update_buyer( 
p_Buyer_ID Buyer.Buyer_ID%TYPE, 
p_BuyerName Buyer.BuyerName%TYPE, 
p_BuyerPhone Buyer.BuyerPhone%TYPE, 
p_Price Buyer.Price%TYPE 
) 
IS 
BEGIN 
UPDATE Buyer 
SET BuyerName = p_BuyerName, 
BuyerPhone = p_BuyerPhone, 
Price = p_Price 
WHERE Buyer_ID = p_Buyer_ID; 
DBMS_OUTPUT.PUT_LINE('Buyer record updated successfully.'); 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
DBMS_OUTPUT.PUT_LINE('Buyer with ID ' || p_Buyer_ID || ' does not exist.'); 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error updating Buyer record: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE update_weather( 
p_Temperature Weather.Temperature%TYPE, 
p_WCondition Weather.WCondition%TYPE, 
p_Humidity Weather.Humidity%TYPE, 
p_Rainfall Weather.Rainfall%TYPE 
) 
IS 
BEGIN 
UPDATE Weather 
SET WCondition = p_WCondition, 
        Humidity = p_Humidity, 
        Rainfall = p_Rainfall 
    WHERE Temperature = p_Temperature; 
     
    DBMS_OUTPUT.PUT_LINE('Weather record updated successfully.'); 
EXCEPTION 
    WHEN NO_DATA_FOUND THEN 
        DBMS_OUTPUT.PUT_LINE('Weather record with temperature ' || p_Temperature || ' does not 
exist.'); 
    WHEN OTHERS THEN 
        DBMS_OUTPUT.PUT_LINE('Error updating Weather record: ' || SQLERRM); 
END; 
 
 
 
CREATE OR REPLACE PROCEDURE update_owner( 
    p_Owner_ID Owner.Owner_ID%TYPE, 
    p_OwnerName Owner.OwnerName%TYPE, 
    p_OwnerAddress Owner.OwnerAddress%TYPE, 
    p_OwnerPhone Owner.OwnerPhone%TYPE 
) 
IS 
BEGIN 
    UPDATE Owner 
    SET OwnerName = p_OwnerName, 
        OwnerAddress = p_OwnerAddress, 
        OwnerPhone = p_OwnerPhone 
    WHERE Owner_ID = p_Owner_ID; 
     
    DBMS_OUTPUT.PUT_LINE('Owner record updated successfully.'); 
EXCEPTION 
    WHEN NO_DATA_FOUND THEN 
        DBMS_OUTPUT.PUT_LINE('Owner record with ID ' || p_Owner_ID || ' does not exist.'); 
    WHEN OTHERS THEN 
        DBMS_OUTPUT.PUT_LINE('Error updating Owner record: ' || SQLERRM); 
END; 
 
 
 
CREATE OR REPLACE PROCEDURE update_fertilizer( 
    p_FertilizerName Fertilizer.FertilizerName%TYPE, 
    p_FertilizerID Fertilizer.FertilizerID%TYPE 
) 
IS 
BEGIN 
    UPDATE Fertilizer 
    SET FertilizerID = p_FertilizerID 
    WHERE FertilizerName = p_FertilizerName; 
     
    DBMS_OUTPUT.PUT_LINE('Fertilizer record updated successfully.'); 
EXCEPTION 
    WHEN NO_DATA_FOUND THEN 
        DBMS_OUTPUT.PUT_LINE('Fertilizer record with name ' || p_FertilizerName || ' does not exist.'); 
    WHEN OTHERS THEN 
        DBMS_OUTPUT.PUT_LINE('Error updating Fertilizer record: ' || SQLERRM); 
END; 
 
 
 
CREATE OR REPLACE PROCEDURE update_production( 
    p_IN_Stock Production.IN_Stock%TYPE, 
    p_Produced Production.Produced%TYPE, 
    p_Seed_ID Production.Seed_ID%TYPE, 
    p_Owner_ID Production.Owner_ID%TYPE, 
    p_Buyer_ID Production.Buyer_ID%TYPE, 
    p_FertilizerName Production.FertilizerName%TYPE, 
    p_Temperature Production.Temperature%TYPE 
) 
IS 
BEGIN 
    UPDATE Production 
    SET Seed_ID = p_Seed_ID, 
        Owner_ID = p_Owner_ID, 
        Buyer_ID = p_Buyer_ID, 
        FertilizerName = p_FertilizerName, 
        Temperature = p_Temperature 
    WHERE IN_Stock = p_IN_Stock 
        AND Produced = p_Produced; 
     
    DBMS_OUTPUT.PUT_LINE('Production record updated successfully.'); 
EXCEPTION 
    WHEN NO_DATA_FOUND THEN 
        DBMS_OUTPUT.PUT_LINE('Production record with IN_Stock ' || p_IN_Stock || ' and Produced ' || 
p_Produced || ' does not exist.'); 
    WHEN OTHERS THEN 
        DBMS_OUTPUT.PUT_LINE('Error updating Production record: ' || SQLERRM); 
END; 
 
 
 
CREATE OR REPLACE PROCEDURE update_crop( 
    crop_id_in IN NUMBER, 
    new_category_in IN VARCHAR2, 
    new_produced_in IN NUMBER 
) 
IS 
BEGIN 
    UPDATE Crop 
    SET CropCategory = new_category_in, 
        Produced = new_produced_in 
    WHERE Crop_ID = crop_id_in; 
 
    DBMS_OUTPUT.PUT_LINE('Crop updated successfully!'); 
EXCEPTION 
    WHEN OTHERS THEN 
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
 
 
 
CREATE OR REPLACE PROCEDURE update_market_info( 
    in_stock_in IN NUMBER, 
    new_price_in IN NUMBER, 
    buyer_id_in IN NUMBER, 
new_address_in IN VARCHAR2 
) 
IS 
BEGIN 
UPDATE MarketInfo 
SET Price = new_price_in, 
MarketAddress = new_address_in 
WHERE In_Stock = in_stock_in 
AND Buyer_ID = buyer_id_in; 
DBMS_OUTPUT.PUT_LINE('Market info updated successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE update_market_info( 
in_stock_in IN NUMBER, 
new_price_in IN NUMBER, 
buyer_id_in IN NUMBER, 
new_address_in IN VARCHAR2 
) 
IS 
BEGIN 
UPDATE MarketInfo 
SET Price = new_price_in, 
MarketAddress = new_address_in 
WHERE In_Stock = in_stock_in 
AND Buyer_ID = buyer_id_in; 
DBMS_OUTPUT.PUT_LINE('Market info updated successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
/ 
begin 
update_market_info(18,4400,10001,'Dhaka'); 
end; 
/ 
select * from marketinfo; 
Deletion Procedures 
CREATE OR REPLACE PROCEDURE delete_seed( 
seed_id_in IN NUMBER 
) 
IS 
BEGIN 
DELETE FROM Seed 
WHERE Seed_ID = seed_id_in; 
DBMS_OUTPUT.PUT_LINE('Seed deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_buyer( 
buyer_id_in IN NUMBER 
) 
IS 
BEGIN 
DELETE FROM Buyer 
WHERE Buyer_ID = buyer_id_in; 
DBMS_OUTPUT.PUT_LINE('Buyer deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_weather( 
temperature_in IN NUMBER 
) 
IS 
BEGIN 
DELETE FROM Weather 
WHERE Temperature = temperature_in; 
DBMS_OUTPUT.PUT_LINE('Weather data deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_owner( 
owner_id_in IN NUMBER 
) 
IS 
BEGIN 
DELETE FROM Owner 
WHERE Owner_ID = owner_id_in; 
DBMS_OUTPUT.PUT_LINE('Owner data deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_fertilizer( 
fertilizer_name_in IN VARCHAR2 
) 
IS 
BEGIN 
DELETE FROM Fertilizer 
WHERE FertilizerName = fertilizer_name_in; 
DBMS_OUTPUT.PUT_LINE('Fertilizer data deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_production( 
in_stock_in IN NUMBER, 
produced_in IN NUMBER 
) 
IS 
BEGIN 
DELETE FROM Production 
WHERE IN_Stock = in_stock_in AND Produced = produced_in; 
DBMS_OUTPUT.PUT_LINE('Production data deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_crop(p_crop_id IN NUMBER) AS 
BEGIN 
DELETE FROM Crop WHERE Crop_ID = p_crop_id; 
DBMS_OUTPUT.PUT_LINE('Crop deleted successfully.'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error deleting crop: ' || SQLERRM); 
END; 
CREATE OR REPLACE PROCEDURE delete_from_marketinfo(in_stock_value NUMBER) AS 
BEGIN 
DELETE FROM MarketInfo WHERE In_Stock = in_stock_value; 
DBMS_OUTPUT.PUT_LINE('Values deleted from MarketInfo table.'); 
EXCEPTION 
WHEN NO_DATA_FOUND THEN 
DBMS_OUTPUT.PUT_LINE('No values found in MarketInfo table for the given In_Stock value.'); 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error occurred: ' || SQLERRM); 
END; 
Retrieval procedures 
CREATE OR REPLACE PROCEDURE show_seed_records AS 
BEGIN 
FOR seed_rec IN (SELECT * FROM Seed)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Seed ID: ' || seed_rec.Seed_ID || ', Seed Name: ' || seed_rec.SeedName); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_buyer_records AS 
BEGIN 
FOR buyer_rec IN (SELECT * FROM Buyer)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Buyer ID: ' || buyer_rec.Buyer_ID || ', Buyer Name: ' || 
buyer_rec.BuyerName ||  
', Buyer Phone: ' || buyer_rec.BuyerPhone || ', Price: ' || buyer_rec.Price); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_weather_records AS 
BEGIN 
FOR weather_rec IN (SELECT * FROM Weather)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Temperature: ' || weather_rec.Temperature || ', Weather Condition: ' || 
weather_rec.WCondition ||  
', Humidity: ' || weather_rec.Humidity || ', Rainfall: ' || weather_rec.Rainfall); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_owner_records AS 
BEGIN 
FOR owner_rec IN (SELECT * FROM Owner)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Owner ID: ' || owner_rec.Owner_ID || ', Owner Name: ' || 
owner_rec.OwnerName ||  
', Owner Address: ' || owner_rec.OwnerAddress || ', Owner Phone: ' || 
owner_rec.OwnerPhone); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_fertilizer_records AS 
BEGIN 
FOR fertilizer_rec IN (SELECT * FROM Fertilizer)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Fertilizer ID: ' || fertilizer_rec.FertilizerID || ', Fertilizer Name: ' || 
fertilizer_rec.FertilizerName); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_production_records AS 
BEGIN 
FOR production_rec IN (SELECT * FROM Production)  
LOOP 
DBMS_OUTPUT.PUT_LINE('In Stock: ' || production_rec.IN_Stock || ', Produced: ' || 
production_rec.Produced ||  
', Owner ID: ' || production_rec.Owner_ID || ', Buyer ID: ' || production_rec.BUYER_ID ||  
', Seed ID: ' || production_rec.Seed_ID || ', Fertilizer Name: ' || 
production_rec.FertilizerName ||  
', Temperature: ' || production_rec.Temperature); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_crop_records AS 
BEGIN 
FOR crop_rec IN (SELECT * FROM Crop)  
LOOP 
DBMS_OUTPUT.PUT_LINE('Crop ID: ' || crop_rec.Crop_ID || ', Crop Category: ' || 
crop_rec.CropCategory ||  
', Produced: ' || crop_rec.Produced); 
END LOOP; 
END; 
CREATE OR REPLACE PROCEDURE show_marketinfo_records AS 
BEGIN\ 
FOR marketinfo_rec IN (SELECT * FROM MarketInfo) LOOP 
DBMS_OUTPUT.PUT_LINE('In_Stock: ' || marketinfo_rec.In_Stock || ' | Price: ' || marketinfo_rec.Price 
|| ' | Buyer_ID: ' || marketinfo_rec.Buyer_ID || ' | MarketAddress: ' || marketinfo_rec.MarketAddress); 
END LOOP; 
END; 
DELETE Procedure: 
CREATE OR REPLACE PROCEDURE delete_owner( 
owner_id_in IN NUMBER 
) 
IS 
BEGIN 
DELETE FROM Owner 
WHERE Owner_ID = owner_id_in; 
DBMS_OUTPUT.PUT_LINE('Owner data deleted successfully!'); 
EXCEPTION 
WHEN OTHERS THEN 
DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM); 
END; 
/ 
begin 
delete_owner(1000); 
end; 
/ 
select * from owner; 
Cursor: 
cursor which counts and displays the 7 record from the above table buyer 
DECLARE 
counter NUMBER := 0; 
buyer_id NUMBER; 
buyer_name VARCHAR2(50); 
CURSOR cur_buyer IS 
SELECT buyer_id, buyer_name FROM buyer; 
BEGIN 
OPEN cur_buyer; 
FETCH cur_buyer INTO buyer_id, buyer_name; 
WHILE cur_buyer%FOUND LOOP 
counter := counter + 1; 
IF counter = 7 THEN 
DBMS_OUTPUT.PUT_LINE('Buyer ID: ' || TO_CHAR(buyer_id) || ', Buyer Name: ' || buyer_name); 
EXIT; 
END IF; 
FETCH cur_buyer INTO buyer_id, buyer_name; 
END LOOP; 
CLOSE cur_buyer; 
END; 
This cursor iterates through all the records in the "buyer" table and increments a counter variable for 
each record. When the counter reaches 7, it displays the buyer ID and name using the PRINT 
statement and breaks out of the loop. 
Triggers: 
Trigger to uppercase owner’s name: 
CREATE OR REPLACE TRIGGER owner_name_uppercase 
BEFORE INSERT OR UPDATE ON Owner 
FOR EACH ROW 
BEGIN 
:NEW.OwnerName := UPPER(:NEW.OwnerName); 
END; 
/ 
CONCLUSION 
To summarize, the Crop Management System (CMS) project is a robust and efficient tool for 
managing digital content. With its versatile features and functionalities, it enables users to 
create, modify, publish, and archive content, as well as manage user roles and permissions. 
The system's reliable database securely stores all the content, which can be easily accessed 
and updated. Additionally, the CMS offers advanced search and filter functions that simplify 
the task of locating and managing specific pieces of content. 
As a result, the CMS is a valuable asset for businesses, organizations, and individuals looking 
to manage digital content more effectively. It streamlines the content management process, 
enhances productivity, and promotes better collaboration among team members. 
