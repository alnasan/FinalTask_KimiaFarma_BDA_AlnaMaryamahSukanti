# FinalTask_KimiaFarma_BDA_AlnaMaryamahSukanti

#-- 1. Produk Terlaris
CREATE OR REPLACE TABLE kimia_farma.produk_terlaris AS
SELECT 
    p.product_id,
    p.product_name,
    COUNT(t.transaction_id) AS jumlah_terjual,
    SUM(t.price * (1 - t.discount_percentage / 100)) AS total_pendapatan
FROM `kimia_farma.kf_final_transaction` t
JOIN `kimia_farma.kf_product` p ON t.product_id = p.product_id
GROUP BY p.product_id, p.product_name
ORDER BY jumlah_terjual DESC
LIMIT 10;
CREATE OR REPLACE VIEW kimia_farma.vw_produk_terlaris AS
SELECT * FROM kimia_farma.produk_terlaris;


#–2. TTransaksi perbulan
CREATE OR REPLACE TABLE kimia_farma.total_transaksi_per_bulan AS
SELECT 
    EXTRACT(YEAR FROM date) AS tahun,
    EXTRACT(MONTH FROM date) AS bulan,
    COUNT(transaction_id) AS total_transaksi,
    SUM(price * (1 - discount_percentage / 100)) AS total_pendapatan
FROM `kimia_farma.kf_final_transaction`
GROUP BY tahun, bulan
ORDER BY tahun, bulan;
CREATE OR REPLACE VIEW kimia_farma.vw_total_transaksi_per_bulan AS
SELECT * FROM kimia_farma.total_transaksi_per_bulan;



#-- 3. Cabang dengan Penjualan Terbaik
CREATE OR REPLACE TABLE kimia_farma.cabang_terbaik AS
SELECT 
    c.branch_id,
    c.branch_name,
    c.kota,
    c.provinsi,
    COUNT(t.transaction_id) AS total_transaksi,
    SUM(t.price * (1 - t.discount_percentage / 100)) AS total_pendapatan
FROM `kimia_farma.kf_final_transaction` t
JOIN `kimia_farma.kf_kantorcabang` c ON t.branch_id = c.branch_id
GROUP BY c.branch_id, c.branch_name, c.kota, c.provinsi
ORDER BY total_pendapatan DESC;
CREATE OR REPLACE VIEW kimia_farma.vw_cabang_terbaik AS
SELECT * FROM kimia_farma.cabang_terbaik;




#-- 4. Perbandingan Stok vs Penjualan
CREATE OR REPLACE TABLE kimia_farma.perbandingan_stok_vs_penjualan AS
SELECT 
    i.branch_id,
    i.product_id,
    i.product_name,
    i.opname_stock AS stok_awal,
    COUNT(t.transaction_id) AS total_terjual
FROM `kimia_farma.kf_inventory` i
LEFT JOIN `kimia_farma.kf_final_transaction` t 
    ON i.branch_id = t.branch_id AND i.product_id = t.product_id
GROUP BY i.branch_id, i.product_id, i.product_name, i.opname_stock
ORDER BY total_terjual DESC;
CREATE OR REPLACE VIEW kimia_farma.vw_perbandingan_stok_vs_penjualan AS
SELECT * FROM kimia_farma.perbandingan_stok_vs_penjualan;



#-- 5. Analisis Laba Bersih & Perbandingan Pendapatan
CREATE OR REPLACE TABLE kimia_farma.analisis_laba_bersih AS
SELECT 
    t.transaction_id,
    t.date,
    c.branch_id,
    c.branch_name,
    c.kota,
    c.provinsi,
    t.customer_name,
    p.product_id,
    p.product_name,
    t.price AS actual_price,
    t.discount_percentage,
    CASE 
        WHEN t.price <= 50000 THEN 0.10 
        WHEN t.price <= 100000 THEN 0.15
        WHEN t.price <= 300000 THEN 0.20
        WHEN t.price <= 500000 THEN 0.25
        ELSE 0.30 
    END AS persentase_gross_laba,
    (t.price * (1 - t.discount_percentage / 100)) AS nett_sales,
    ((t.price * (1 - t.discount_percentage / 100)) * 
    CASE 
        WHEN t.price <= 50000 THEN 0.10 
        WHEN t.price <= 100000 THEN 0.15
        WHEN t.price <= 300000 THEN 0.20
        WHEN t.price <= 500000 THEN 0.25
        ELSE 0.30 
    END) AS nett_profit
FROM `kimia_farma.kf_final_transaction` t
JOIN `kimia_farma.kf_product` p ON t.product_id = p.product_id
JOIN `kimia_farma.kf_kantorcabang` c ON t.branch_id = c.branch_id;
CREATE OR REPLACE VIEW kimia_farma.vw_analisis_laba_bersih AS
SELECT * FROM kimia_farma.analisis_laba_bersih;



#-- 6. top 10 transaksi cabang provinsi
CREATE OR REPLACE TABLE kimia_farma.top_10_transaksi_cabang_provinsi
AS
SELECT
kc.provinsi,
kc.branch_name,
COUNT(ft.transaction_id) AS total_transaksi
FROM kimia_farma.kf_final_transaction ft
JOIN kimia_farma.kf_kantorcabang kc
ON ft.branch_id=kc.branch_id
GROUP BY 1, 2
ORDER BY total_transaksi DESC
LIMIT 10;
CREATE OR REPLACE VIEW kimia_farma.
vw_top_10_transaksi_cabang_provinsi AS
SELECT * FROM kimia_farma_analysis.top_10_transaksi_cabang_provinsi;


#-- 7. Top 5 Cabang dengan Rating Tertinggi, Namun Rating Transaksi Terendah (Data Statis)
CREATE OR REPLACE TABLE kimia_farma.top_5_rating_cabang
AS
SELECT 
    kc.branch_id,
    kc.branch_name,
    kc.provinsi,
    kc.rating AS rating_cabang,
    AVG(ft.rating) AS rating_transaksi
FROM `kimia_farma.kf_final_transaction` ft
JOIN `kimia_farma.kf_kantorcabang` kc
ON ft.branch_id = kc.branch_id
GROUP BY kc.branch_id, kc.branch_name, kc.provinsi, kc.rating
ORDER BY kc.rating DESC, rating_transaksi ASC
LIMIT 5;
CREATE OR REPLACE VIEW kimia_farma.vw_top_5_rating_cabang AS
SELECT * FROM kimia_farma.top_5_rating_cabang;


#-- 8. top_10_nett_sales_cabang_provinsi
CREATE OR REPLACE TABLE kimia_farma.top_10_nett_sales_cabang_provinsi AS
SELECT 
    kc.provinsi, 
    kc.branch_name, 
    SUM(ft.price * (1 - ft.discount_percentage / 100)) AS nett_sales
FROM `kimia_farma.kf_final_transaction` ft
JOIN `kimia_farma.kf_kantorcabang` kc 
ON ft.branch_id = kc.branch_id
GROUP BY 1, 2
ORDER BY nett_sales DESC
LIMIT 10;
CREATE OR REPLACE VIEW kimia_farma.vw_top_10_nett_sales_cabang_provinsi AS
SELECT * FROM kimia_farma.top_10_nett_sales_cabang_provinsi;



#--9 CREATE OR REPLACE TABLE kimia_farma.geo_map_total_profit_per_provinsi AS
SELECT 
    kc.provinsi,
    SUM(ft.price * (1 - ft.discount_percentage / 100) * 
        CASE 
            WHEN ft.price <= 50000 THEN 0.1
            WHEN ft.price > 50000 AND ft.price <= 100000 THEN 0.15
            WHEN ft.price > 100000 AND ft.price <= 300000 THEN 0.2
            WHEN ft.price > 300000 AND ft.price <= 500000 THEN 0.25
            ELSE 0.3
        END
    ) AS total_profit
FROM `kimia_farma.kf_final_transaction` ft
JOIN `kimia_farma.kf_kantorcabang` kc 
ON ft.branch_id = kc.branch_id
GROUP BY 1;
CREATE OR REPLACE VIEW kimia_farma.vw_geo_map_total_profit_per_provinsi AS
SELECT * FROM kimia_farma.geo_map_total_profit_per_provinsi;
Link : https://console.cloud.google.com/bigquery?sq=559559882285:9c7b2cd798544b25a378541b73390e24



#-–10
CREATE OR REPLACE TABLE kimia_farma.summary_dashboard AS
SELECT 
    COUNT(DISTINCT ft.transaction_id) AS total_transaksi,
    SUM(ft.price * (1 - ft.discount_percentage / 100)) AS total_pendapatan,
    AVG(ft.discount_percentage) AS rata_rata_diskon,
    AVG(ft.rating) AS rata_rata_rating
FROM `kimia_farma.kf_final_transaction` ft;
CREATE OR REPLACE VIEW kimia_farma.vw_summary_dashboard AS
SELECT * FROM kimia_farma.summary_dashboard;
