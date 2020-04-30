# [Cold-Turkey-Navigating-Guidance-Withdrawal-with-Supply-Chain-Data](https://www.spglobal.com/marketintelligence/en/news-insights/research/quantamental-research-brief-cold-turkey-navigating-guidance-withdrawal-with-supply-chain-data)

---

**1.** Retrieve Panjiva U.S. shipment data associated with Russell 3000 companies:

```sql
WITH r3k AS (
SELECT DISTINCT cs.companyid, si.simpleIndustryDescription
FROM ciqindexconstituent cic
JOIN ciqtradingitem cti
ON cic.tradingitemid = cti.tradingitemid
JOIN ciqsecurity cs
ON cs.securityid = cti.securityid
JOIN ciqcompany co ON co.companyid = cs.companyid
JOIN ciqSimpleIndustry si ON si.simpleIndustryId = co.simpleIndustryId
WHERE indexid = 2668795 --russell3
AND cic.todate is NULL
), imports AS (
SELECT * FROM XFL_PANJIVA.dbo.panjivaUSImport2016
UNION ALL
SELECT * FROM XFL_PANJIVA.dbo.panjivaUSImport2017
UNION ALL
SELECT * FROM XFL_PANJIVA.dbo.panjivaUSImport2018
UNION ALL
SELECT * FROM XFL_PANJIVA.dbo.panjivaUSImport2019
UNION ALL
SELECT * FROM XFL_PANJIVA.dbo.panjivaUSImport2020
)
SELECT datepart(year, arrivalDate) AS yr, datepart(month, arrivalDate) AS mo,
sum(volumeTEU) AS teu, r3k.companyid, r3k.simpleIndustryDescription
FROM imports
JOIN XFL_PANJIVA.dbo.panjivaCompanyCrossRef ccr
ON ccr.identifierValue = imports.conPanjivaId
JOIN ciqcompanyultimateparent ult
ON ult.companyId = ccr.companyId
JOIN r3k
ON r3k.companyid = ult.companyId
GROUP BY datepart(year, arrivalDate), datepart(month, arrivalDate),
r3k.companyid, r3k.simpleIndustryDescription
```

**2.** Retrieve Russell 3000 Guidance:
```sql
SELECT situation, announcedDate, keydevEventTypeName, co.companyId,
companyName, country, si.simpleIndustryDescription, kd.keyDevId
FROM ciqKeyDevToObjectToEventType kdo
JOIN ciqKeyDev kd
ON kd.keyDevId = kdo.keyDevId
JOIN ciqKeyDevObjectRoleType kdr
ON kdr.keyDevToObjectRoleTypeId = kdo.keyDevToObjectRoleTypeId
JOIN ciqKeyDevEventType kde ON kde.keyDevEventTypeId = kdo.keyDevEventTypeId
JOIN ciqCompany co ON co.companyId = kdo.objectId
JOIN ciqCountryGeo cty ON cty.countryId = co.countryId
JOIN ciqSimpleIndustry si ON si.simpleIndustryId = co.simpleIndustryId
WHERE kde.keyDevEventTypeId IN (26, 27, 29, 49, 225)
AND announcedDate >= '2018-01-01'
```
