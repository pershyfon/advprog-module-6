## Reflection
### Commit 1: handle_connection method
Method `handle_connection` bertanggung jawab untuk membaca steam line TCP yang masuk menggunakan BufReader baris demi baris sampai menemukan baris kosong, yang menandakan akhir dari header HTTP Request. Kemudian mencetak baris-baris yang terkumpul sebagai Request.