# Làm sạch dữ liệu mua bán nhà 

# Tổng quan
1. Tên dự án
2. Mục tiêu
3. Nguồn dữ liệu
4. Công cụ
5. Code

# 1. Tên dự án:

**Làm sạch dữ liệu mua bán nhà**

# 2. Mục tiêu:

Dự án nhằm làm sạch dữ liệu để giúp việc mapping dữ liệu thuận tiện hơn thông qua: định dạng dữ liệu, phân tách trường, điền dữ liệu, xóa dữ liệu trùng lặp, đồng bộ hóa giá trị dữ liệu.

# 3. Nguồn dữ liệu:

Dữ liệu về doanh số bán nhà tại Nashville trong giai đoạn 2013-2019 với quy mô dữ liệu là 56,374 bản ghi.
- *Nguồn dữ liệu: [Alex The Analyst](https://github.com/AlexTheAnalyst/PortfolioProjects/blob/main/Nashville%20Housing%20Data%20for%20Data%20Cleaning.xlsx)*

# 4. Công cụ:

- SQL:
  * Câu lệnh sử dụng: JOIN, UPDATE, SUBSTRING, PARSENAME, WINDOW FUNCTION, CTE.

# 5. Code
	
SELECT * 
FROM Houseselling

-- TẠO BẢNG TẠM ĐỂ THỰC HIỆN LÀM SẠCH DỮ LIỆU

	SELECT *
	INTO ##DATA
	FROM Houseselling

-- 1.CHUẨN HÓA KIỂU DỮ LIỆU NGÀY

	ALTER TABLE ##DATA
		ADD Sale_date DATE 

	UPDATE ##DATA
	SET Sale_date = CONVERT (DATE, SaleDate)

-- 2. BỔ SUNG DỮ LIỆU ĐỊA CHỈ BỊ THIẾU
	
	UPDATE A
	SET A.PropertyAddress = B.PropertyAddress
	FROM ##DATA A
	JOIN ##DATA B ON A.ParcelID = B.ParcelID
	WHERE B.PropertyAddress IS NOT NULL

-- 3. TÁCH GIÁ TRỊ CỦA CỘT ĐỊA CHỈ THÀNH CÁC CỘT RIÊNG BIỆT (ĐỊA CHỈ, THÀNH PHỐ, BANG) 

----- TÁCH CỘT PROPERTY ADDRESS

	ALTER TABLE ##DATA
		ADD [PropertyAdress] NVARCHAR(255)

	UPDATE ##DATA
	SET [PropertyAdress] = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 )
	
	ALTER TABLE ##DATA
		ADD PropertyCity NVARCHAR(255)

	UPDATE ##DATA
	SET PropertyCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))
	
----- TÁCH CỘT OWNERADDRESS 

	ALTER TABLE ##DATA
		ADD Owneraddress_split NVARCHAR(255)
	
	UPDATE ##DATA
	SET Owneraddress_split = PARSENAME(REPLACE(OWNERADDRESS,',','.'),3)

	ALTER TABLE ##DATA
		ADD Ownercity_split NVARCHAR(255)
	
	UPDATE ##DATA
	SET Ownercity_split = PARSENAME(REPLACE(OWNERADDRESS,',','.'),2)

	ALTER TABLE ##DATA
		ADD Ownerstate_split NVARCHAR(255)
	
	UPDATE ##DATA
	SET Ownerstate_split = PARSENAME(REPLACE(OWNERADDRESS,',','.'),1)

-- 4. ĐỒNG BỘ GIÁ TRỊ DỮ LIỆU: THAY ĐỔI GIÁ TRỊ 'YES' VÀ 'NO' THÀNH 'Y' VÀ 'N'

	UPDATE ##DATA
	SET SoldAsVacant = CASE		
				WHEN SoldAsVacant = 'Yes' THEN 'Y'
				WHEN SoldAsVacant = 'No' THEN 'N'
				ELSE SoldAsVacant
			END

-- 5. LOẠI BỎ GIÁ TRỊ TRÙNG LẶP

	WITH ROWNUM AS (
	SELECT *,
			ROW_NUMBER() OVER (	
					PARTITION BY ParcelID,
							PropertyAddress,
							SalePrice,
							Saledate,
							LegalReference
					ORDER BY UniqueID
						) row_num

	FROM ##DATA
	)
	DELETE 
	FROM ROWNUM
	WHERE row_num > 1

-- 6. XÓA DỮ LIỆU KHÔNG SỬ DỤNG
	
	ALTER TABLE ##DATA
		DROP COLUMN PropertyAddress,OwnerAddress, TaxDistrict, saledate

-- TẠO BẢNG DỮ LIỆU ĐÃ ĐƯỢC LÀM SẠCH
	
	DROP TABLE IF EXISTS HOUSESELLING_UPDATE
	SELECT *
	INTO HOUSESELLING_UPDATE
	FROM ##DATA
