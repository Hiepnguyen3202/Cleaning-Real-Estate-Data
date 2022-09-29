/*
Cleaning Real Estate Data in MS SQL 
*/


Select *  --- Have a first glance of data
From [Data Cleaning Project].[dbo].[NashvilleHousing]

--------------------------------------------------------------------------------------------------------------------------

--- Standardize Date Format ---


SELECT SaleDate, CONVERT(DATE,SaleDate)
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]


ALTER TABLE NashvilleHousing --- Add a column
ADD SaleDateConverted DATE;

UPDATE NashvilleHousing --- Check changes
SET SaleDateConverted = CONVERT(DATE,SaleDate)

SELECT *
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]

--------------------------------------------------------------------------------------------------------------------------

--- Populate Property Address Data ---


SELECT * --- Check there are null PropertyAddress?
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]
WHERE PropertyAddress IS NULL
 --- We found that there are some houses with null address. Need to find these address

SELECT DISTINCT(ParcelID) --- Find distinct ParcelID
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]

SELECT DISTINCT(ParcelID), COUNT(*) AS num --- Find some houses with same ParcelID
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]
GROUP BY ParcelID
ORDER BY num DESC

SELECT * --- Check randomly 1 ParcelID to see whether these parcels have the same address
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]
WHERE REPLACE(ParcelID,' ', '') LIKE '034070B015.00'

--- We can see that the same ParcelID will share the same PropertyAddress

UPDATE a --- Update the information about PropertyAddress for houses with null value before
SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
FROM [Data Cleaning Project].[dbo].[NashvilleHousing] AS a
JOIN [Data Cleaning Project].[dbo].[NashvilleHousing] AS b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL


--------------------------------------------------------------------------------------------------------------------------

--- Breaking out Address into Individual Columns (Address, City, State) ---

SELECT PropertyAddress --- Look for adrress column, each address has a specific address plus city name
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]


-- Split the address and city into 2 columns
SELECT SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) AS Address  -- add -1 to get rid of ','
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]

SELECT SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) AS City  
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]


-- Add Two new columns
ALTER TABLE NashvilleHousing
ADD PropertySplitAddress Nvarchar(255)

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)

ALTER TABLE NashvilleHousing
ADD PropertySplitCity Nvarchar(255)

UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))

SELECT *  --- Check table
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]




--------------------------------------------------------------------------------------------------------------------------


--- Change Y and N to Yes and No in "Sold as Vacant" field ---

SELECT *  --- Check table
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]

SELECT DISTINCT(SoldASVacant), COUNT(*) --- check values of SoldAsVacant column
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]
GROUP BY SoldASVacant

-- Use Case statement to change Y and N to Yes and No
SELECT SoldASVacant,
  CASE WHEN SoldASVacant = 'Y' THEN 'Yes'
       WHEN SoldASVacant = 'N' THEN 'No'
	   ELSE SoldASVacant
	   END
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]

UPDATE NashvilleHousing
SET SoldAsVacant = (CASE WHEN SoldASVacant = 'Y' THEN 'Yes'
       WHEN SoldASVacant = 'N' THEN 'No'
	   ELSE SoldASVacant
	   END)

SELECT DISTINCT(SoldASVacant), COUNT(*) --- check values of SoldAsVacant column again
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]
GROUP BY SoldASVacant
-- It works



-----------------------------------------------------------------------------------------------------------------------------------------------------------

--- Remove Duplicates ---

-- USE CTE, Window function to delete duplicates
WITH cte AS (SELECT*, ROW_NUMBER() OVER (PARTITION BY ParcelID,
                                         PropertyAddress,
										 SalePrice,
										 SaleDate,
										 LegalReference,
										 OwnerName,
										 Acreage
										 ORDER BY UniqueID) AS row_num
FROM [Data Cleaning Project].[dbo].[NashvilleHousing])

DELETE
FROM cte
WHERE row_num > 1 --- We can see there are so many duplicate rows




---------------------------------------------------------------------------------------------------------

--- Delete Unused Columns ---

SELECT *  --- Check table
FROM [Data Cleaning Project].[dbo].[NashvilleHousing]

ALTER TABLE NashvilleHousing
DROP COLUMN PropertyAddress, SaleDate, OwnerAddress
