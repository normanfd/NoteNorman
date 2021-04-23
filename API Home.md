### Logic API Home

***

**Request**

SkTime = bulan yg berlaku -- YYYYMMDD

BdTIme = hari saat ini -- YYYYMMDD

**MaxDate**

maxDate = tanggal BDtime - tanggal akhir bulan SKtime 

- max date >  0, maka max date = tanggal akhir bulan SKtime
- max date < 0, maka max date = bdTime

**Var**

| Nama Variabel        | Perhitungan                         |
| -------------------- | ----------------------------------- |
| @StartCurrent        | awal bulan dari max date            |
| @EndCurrent          | maxdate                             |
| @StartLastMonth      | awal bulan dari (maxdate - 1 bulan) |
| @EndLastMonth        | maxdate - 1 bulan                   |
| @Last100DayCurrent   | maxdate - 100 hari                  |
| @Last100DayLastMonth | maxdate - 1 bulan -100 hari         |

#### Perhitungan total

| No   | Nama                 | Perhitungan                                                  | ket                                 |
| ---- | -------------------- | ------------------------------------------------------------ | ----------------------------------- |
| 1    | Total Current Lead   | SK_created_date_asset antara @StartCurrent dan @EndCurrent   | ikut SP                             |
| 2    | TotalLastLead        | SK_Created_Date_Asset antara @StartLastMonth dan @EndLastMonth | ikut SP                             |
| 3    | TotalCurrentProspect | SK_Closing_Date antara @StartCurrent dan @EndCurrent         | ikut SP                             |
|      |                      | SK_Created_Date_Asset antara @StartCurrent dan @EndCurrent   | ikut SP                             |
| 4    | TotalLastProspect    | SK_Closing_Date antara @StartLastMonth dan @EndLastMonth     | ikut SP                             |
|      |                      | SK_Created_Date_Asset antara @StartLastMonth dan @EndLastMonth | ikut SP                             |
| 5    | TotalCurrentNtf      | SK_Golive_Date antara @StartCurrent dan @EndCurrent          | ikut SP perhitungan disburse Amount |
|      |                      | SK_Created_Date_Asset @Last100DayCurrent dan @EndCurrent     |                                     |
| 6    | TotalLastNtf         | SK_Golive_Date antara @StartLastMonth dan @EndLastMonth      | ikut SP                             |
|      |                      | SK_Created_Date_Asset antara @Last100DayLastMonth dan @EndLastMonth | ikut SP                             |
| 7    | TotalCurrentGolive   | SK_Golive_Date antara @StartCurrent dan @EndCurrent          |                                     |
| 8    | TotalLastGolive      | SK_Golive_Date antara @StartLastMonth dan @EndLastMonth      |                                     |

#### Example

```
startcurrent               = 1/04/2021
endcurrent                = 16/04/2021
Last100DayCurrent = endcurrent  - 100 hari 
Amount Disburse
```

#### RAW

```sql
SELECT 
	 Product_Asset_condition ProductAssetCondition,
	 SUM(CASE WHEN SK_Created_Date_Asset BETWEEN CONVERT(int, @StartCurrent) AND CONVERT(int, @EndCurrent) THEN 1 ELSE 0 END) TotalCurrentLead,
	 SUM(CASE WHEN SK_Created_Date_Asset BETWEEN CONVERT(int, @StartLastMonth) AND CONVERT(int, @EndLastMonth) THEN 1 ELSE 0 END) TotalLastLead,	 
	 SUM(CASE WHEN (SK_Closing_Date BETWEEN CONVERT(int, @StartCurrent) AND CONVERT(int, @EndCurrent)) AND (SK_Created_Date_Asset BETWEEN CONVERT(int, @StartCurrent) and CONVERT(int, @EndCurrent)) THEN 1 ELSE 0 END) TotalCurrentProspect,
	 SUM(CASE WHEN (SK_Closing_Date BETWEEN CONVERT(int, @StartLastMonth) AND CONVERT(int, @EndLastMonth)) AND (SK_Created_Date_Asset BETWEEN CONVERT(int, @StartLastMonth) and CONVERT(int, @EndLastMonth)) THEN 1 ELSE 0 END) TotalLastProspect,
	 SUM(CASE WHEN (SK_Golive_Date BETWEEN CONVERT(int, @StartCurrent) AND CONVERT(int, @EndCurrent)) AND (SK_Created_Date_Asset BETWEEN CONVERT(int, @Last100DayCurrent) and CONVERT(int, @EndCurrent)) THEN Booking_Amount_Last100Day ELSE 0 END) TotalCurrentAmountFunding,
	 SUM(CASE WHEN (SK_Golive_Date BETWEEN CONVERT(int, @StartLastMonth) AND CONVERT(int, @EndLastMonth)) AND (SK_Created_Date_Asset BETWEEN CONVERT(int, @Last100DayLastMonth) and CONVERT(int, @EndLastMonth)) THEN Booking_Amount_Last100Day ELSE 0 END) TotalLastAmountFunding,
	 SUM(CASE WHEN (SK_Golive_Date BETWEEN CONVERT(int, @StartCurrent) AND CONVERT(int, @EndCurrent)) AND (SK_Created_Date_Asset BETWEEN CONVERT(int, @Last100DayCurrent) and CONVERT(int, @EndCurrent)) THEN NTF ELSE 0 END) TotalCurrentNtf,
	 SUM(CASE WHEN (SK_Golive_Date BETWEEN CONVERT(int, @StartLastMonth) AND CONVERT(int, @EndLastMonth)) AND (SK_Created_Date_Asset BETWEEN CONVERT(int, @Last100DayLastMonth) and CONVERT(int, @EndLastMonth)) THEN NTF ELSE 0 END) TotalLastNtf
FROM 
	Dashboard_Digital_Sales_Funnel_New WITH(NOLOCK)
WHERE SORID = @SorId
GROUP BY Product_Asset_condition
```

#### RAW SP

```sql
USE [AISDB]
GO

/****** Object:  StoredProcedure [dbo].[DashBoardDigital_Box]    Script Date: 16/04/2021 10:56:12 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO


CREATE Proc [dbo].[DashBoardDigital_Box] --[DashBoardDigital_Box] '1705NC0002','20181201', '20181206','All'
-- adi 26 Nov 2018 - 201804CR021
@MarketingID varchar(20),
@SKTime varchar(20),
@BDTime varchar(20),
@Product varchar(30)
AS
BEGIN

declare @whereCond varchar(300)

if @MarketingID = 'All'
begin

set @whereCond = ' '

end

else
begin
set @whereCond = ' and SORID ='''+@MarketingID+''''

end

if @Product <> 'All'
begin
	set @whereCond = @whereCond + ' and Product_Asset_condition ='''+@Product+''''
end

Declare @Time varchar(20), @StartCurrent varchar(20), @EndCurrent varchar(20), @StartLast varchar(20), @EndLast varchar(20), @100Time varchar(20), @100LastTime varchar(20)

if datediff(day,dateadd(day,-1,dateadd(month,1,@SKTime)), @BDTime) >= 0
begin
	
	set @Time = convert(varchar(10),dateadd(day,-1,dateadd(month,1,@SKTime)),112)
end
else
begin
	set @Time = @BDTime
end
set @StartCurrent =  convert(varchar(10),DATEADD(day, -day(@Time)+1,@Time),112)
set @EndCurrent =  @Time
set @StartLast =  convert(varchar(10),DATEADD(month, -1,DATEADD(day, -day(@Time)+1,@Time)),112)
set @EndLast =  convert(varchar(10),DATEADD(month, -1,@Time),112)
set @100Time = convert(varchar(10),DATEADD(day, -100, @EndCurrent),112)
set @100LastTime = convert(varchar(10),DATEADD(day, -100, @EndLast),112)	


exec('
Declare @TotalLead int,
	@PercentageLead int,
	@TotalProspect int,
	@PercentageProspect int,
	@TotalBooking int,
	@PercentageBooking int,
	@TotalAmountFunding numeric(17,2),
	@PercentageAmountFunding int,
	@TotalInput int,
	@TotalBooking2 int

select 
	@TotalLead = TotalCurrent,
	@PercentageLead = case when convert(numeric(17,2),TotalLast) = 0
		then 0
		else convert(int,convert(numeric(17,2),(TotalCurrent-TotalLast))/convert(numeric(17,2),TotalLast) * 100) end 
 from (
select 
	''Total Leads'' Name,
	 SUM(case when SK_Created_Date_Asset between convert(int,'''+@StartCurrent+''') AND convert(int,'''+@EndCurrent+''')
			then 1
		else 0 end) TotalCurrent,
	 SUM(case when SK_Created_Date_Asset between convert(int,'''+@StartLast+''') AND convert(int,'''+@EndLast+''')
			then 1
		else 0 end) TotalLast		 
 from [S_DWPORTAL].DWH.dbo.Dashboard_Digital_Sales_Funnel_New
where SK_Created_Date_Asset between convert(int,'''+@StartLast+''') and convert(int,'''+@EndCurrent+''') ' + @WhereCond + '
) aa


 
select 
	@TotalProspect = TotalCurrent,
	@PercentageProspect = case when convert(numeric(17,2),TotalLast) = 0
		then 0
		else convert(int,convert(numeric(17,2),(TotalCurrent-TotalLast))/convert(numeric(17,2),TotalLast) * 100) end 
 from (
select 
	''Prospect'' Name,
	 SUM(case when (SK_Closing_Date between convert(int,'''+@StartCurrent+''') AND convert(int,'''+@EndCurrent+'''))
				AND (SK_Created_Date_Asset between convert(int,'''+@StartCurrent+''') and convert(int,'''+@EndCurrent+'''))
			then 1
		else 0 end) TotalCurrent,
	 SUM(case when (SK_Closing_Date between convert(int,'''+@StartLast+''') AND convert(int,'''+@EndLast+'''))
			AND (SK_Created_Date_Asset between convert(int,'''+@StartLast+''') and convert(int,'''+@EndLast+'''))
			then 1
		else 0 end) TotalLast		 
 from [S_DWPORTAL].DWH.dbo.Dashboard_Digital_Sales_Funnel_New
where SK_Created_Date_Asset between convert(int,'''+@StartLast+''') and convert(int,'''+@EndCurrent+''')
and SK_Closing_Date between convert(int,'''+@StartLast+''') and convert(int,'''+@EndCurrent+''')  ' + @WhereCond + '
) aa

 
select 
	@TotalInput = TotalCurrent
 from (
select 
	''Input'' Name,
	 SUM(case when SK_NewApplication_Date between convert(int,'''+@StartCurrent+''') AND convert(int,'''+@EndCurrent+''')
			then 1
		else 0 end) TotalCurrent	 
 from [S_DWPORTAL].DWH.dbo.Dashboard_Digital_Sales_Funnel_New
where SK_Created_Date_Asset between convert(int,'''+@StartCurrent+''') and convert(int,'''+@EndCurrent+''')
and SK_NewApplication_Date between convert(int,'''+@StartCurrent+''') and convert(int,'''+@BDTime+''')  ' + @WhereCond + '
) aa

 
select 
	@TotalBooking = TotalCurrent,
	@PercentageBooking = case when convert(numeric(17,2),TotalLast) = 0
		then 0
		else convert(int,convert(numeric(17,2),(TotalCurrent-TotalLast))/convert(numeric(17,2),TotalLast) * 100) end 
 from (
select 
	''Booking'' Name,
	  SUM(case when (SK_Golive_Date between convert(int,'''+@StartCurrent+''') AND convert(int,'''+@EndCurrent+'''))
		AND (SK_Created_Date_Asset between convert(int,'''+@100Time+''') and convert(int,'''+@EndCurrent+'''))
			then 1
		else 0 end) TotalCurrent,
	  SUM(case when (SK_Golive_Date between convert(int,'''+@StartLast+''') AND convert(int,'''+@EndLast+'''))
			AND (SK_Created_Date_Asset between convert(int,'''+@100LastTime+''') and convert(int,'''+@EndLast+'''))
			then 1
		else 0 end) TotalLast		 
 from [S_DWPORTAL].DWH.dbo.Dashboard_Digital_Sales_Funnel_New
where SK_Created_Date_Asset between convert(int,'''+@100LastTime+''') and convert(int,'''+@EndCurrent+''')
and SK_Golive_Date between convert(int,'''+@StartLast+''') and convert(int,'''+@EndCurrent+''')  ' + @WhereCond + '
) aa

 
select 
	@TotalBooking2 = TotalCurrent
 from (
select 
	''Booking2'' Name,
	 SUM(case when SK_Golive_Date between convert(int,'''+@StartCurrent+''') AND convert(int,'''+@EndCurrent+''')
			then 1
		else 0 end) TotalCurrent	 
 from [S_DWPORTAL].DWH.dbo.Dashboard_Digital_Sales_Funnel_New
where SK_Created_Date_Asset between convert(int,'''+@StartCurrent+''') and convert(int,'''+@EndCurrent+''')
and SK_Golive_Date between convert(int,'''+@StartCurrent+''') and convert(int,'''+@EndCurrent+''') ' + @WhereCond + '
) aa

 
select 
	@TotalAmountFunding = TotalCurrent,
	@PercentageAmountFunding = case when convert(numeric(17,2),TotalLast) = 0
		then 0
		else convert(int,convert(numeric(17,2),(TotalCurrent-TotalLast))/convert(numeric(17,2),TotalLast) * 100) end
 from (
select 
	''Amount Funding'' Name,
	 SUM(case when (SK_Golive_Date between convert(int,'''+@StartCurrent+''') AND convert(int,'''+@EndCurrent+'''))
		AND (SK_Created_Date_Asset between convert(int,'''+@100Time+''') and convert(int,'''+@EndCurrent+'''))
			then Booking_Amount_Last100Day
		else 0 end) TotalCurrent,
	  SUM(case when (SK_Golive_Date between convert(int,'''+@StartLast+''') AND convert(int,'''+@EndLast+'''))
			AND (SK_Created_Date_Asset between convert(int,'''+@100LastTime+''') and convert(int,'''+@EndLast+'''))
			then Booking_Amount_Last100Day
		else 0 end) TotalLast		 
 from [S_DWPORTAL].DWH.dbo.Dashboard_Digital_Sales_Funnel_New
where SK_Created_Date_Asset between convert(int,'''+@100LastTime+''') and convert(int,'''+@EndCurrent+''')
and SK_Golive_Date between convert(int,'''+@StartLast+''') and convert(int,'''+@EndCurrent+''') ' + @WhereCond + '
) aa

select @TotalLead TotalLead,
	@PercentageLead PercentageLead,
	@TotalProspect TotalProspect,
	@PercentageProspect PercentageProspect,
	@TotalBooking TotalBooking,
	@PercentageBooking PercentageBooking,
	@TotalAmountFunding TotalAmountFunding,
	@PercentageAmountFunding PercentageAmountFunding,
	@TotalInput - @TotalBooking2 TotalPending

')


END



GO
```

