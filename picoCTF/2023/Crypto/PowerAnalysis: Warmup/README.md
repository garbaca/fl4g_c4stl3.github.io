Mở challenge và chạy instance, ta có file encrypt.py.

Chạy instance,ta nhận được số leaked bit.PowerAnalysis: Warmup

Đọc kỹ file encrypt.py, nhận ra số leaked bit này là số bit trong của Sbox[key^plaintext], với plaintext là 1 dãy hex 16 byte (32 ký tự) và key cũng 16 byte (và cũng là flag p tìm). 

Lưu ý, Sbox[a^b] = Sbox[c] (c=a^b) là giá trị ở dòng chứa ký tự thứ nhất và cột chứa ký tự thứ 2. Chẳng hạn, Sbox[a^b]=Sbox[6c] thì dò trong bảng Sbox tìm giá trị ở dòng 6 cột 12 (c=12) (đếm cột và dòng từ 1 nhé!)

Bây giờ, đến lúc triệu hồi ma pháp và tưởng tượng nào. Hãy hình dung, khi ta chọn plaintext="00000000000000000000000000000000", thì giá trị trả về cũng chính là số các byte mà LSB(Sbox[byte[i]]) = 1, i chạy từ 1 đến 16 nhé (LSB là least significant bit, vd '0001' thì LSB=1, còn '010' thì LSB=0)

Hãy đếm byte từ trái qua phải, tử 1 đến 16, mỗi byte chứa 2 ký tự hexa (ký tự hexa là từ 0 đến f).

Hãy gọi số các leaked bit khi nhận plaintext="00000000000000000000000000000000" là N.

H giả dụ xét byte số 1, và ta tiến hành nhập các plaintext có dạng "xx000000000000000000000000000000", khi đó sẽ có 2 trường hợp xảy ra:
+> Trường hợp 1: Có những plaintext xuất ra số leaked bit = N, những plaintext còn lại xuất ra số leaked bit > N
+> Trường hợp 2: Có những plaintext xuất ra số leaked bit = N, những plaintext còn lại xuất ra số leaked bit < N

Lưu ý cẩn trọng: Ở các byte trong key xor vs byte "00" sẽ ra số leaked bit của chính byte đó trong key, vì LSB(Sbox[byte[i]^"00"]) = LSB(Sbox[byte[i]])
Vả N=Sigma(LSB(byte[i])), i chạy tử 1 tới 16, nên sự thay đổi của số leaked byte sẽ phụ thuộc vào byte ta đang khảo sát, trong trường hợp này là byte số 1.

Nên ở vd trên khi xét byte thứ 1, ta chỉ cho byte 1 thay đổi giá trị mà th, các byte còn lại gán = "00" để xor ra chính byte trong cái key, đảm bảo cho việc xác định chính xác LSB(byte[1]) = 0 hay 1 ở dưới này.

Hãy hình dung mà xem, ở trường hợp 1, nếu byte thứ nhất có LSB(Sbox[byte[1]]) = 0 thì khi nhận plaintext mà xor thì ở Sbox[byte[1]^plaintext], LSB(Sbox[byte[1]^plaintext]) sẽ có 2 trường hơp là 1 và 0. Nếu  LSB(Sbox[byte[1]]) = 0 và LSB(Sbox[byte[1]^plaintext]) = 0  thì dĩ nhiên số leaked bit trả về sẽ bằng N, còn khi LSB(Sbox[byte[1]^plaintext]) = 1 thì số leaked bit sẽ lớn hơn N (=N+1).

Còn ở trường hợp 2, nếu byte thứ nhất có LSB(Sbox[byte[1]])=1 thì khi nhận plaintext mà xor thì ở Sbox[byte[1]^plaintext], LSB(Sbox[byte[1]^plaintext]) sẽ có 2 trường hơp là 1 và 0. Nếu LSB(Sbox[byte[1]])=1 và LSB(Sbox[byte[1]^plaintext]) = 0 r thì dĩ nhiên số leaked bit trả về sẽ p bé hơn hơn N (=N-1), còn khi leaked bit trả về là 1 thì số leaked bit sẽ bằng N.

Lưu ý vì sao bt là N-1 hay N+1 là do ta chỉ xét sự thay đổi của số leaked bit khi thay đổi các giá trị plaintext tại byte nhất định để kiểm tra LSB ở byte đó trong key là 1 hay 0, còn các byte khác của plaintext gán "00" nên khi xor vs byte tương ứng trong key sẽ không gây thay đổi gì nên leaked bit của các byte đó sẽ 0 đổi. Sự thay đổi chỉ được gây ra ở byte đang xét (byte 0 được xor với "00").
Các byte còn lại xét y chang.

Do đó: Hãy xét từng byte 1, mỗi byte xor chừng tổi thiểu 17 plaintext trở lên, muốn an toàn thì 50 cũng đc. Xét byte nào, thì plaintext được xor p thay đổi giá trị tương ứng ở byte đó mà th,các  byte còn lại phải giữ giá trị "00" nhé.
VD: key ^ "00000000000000000000000000000000" ra số leaked bit = 6
Xét byte đầu tiên, sinh bộ plaintext = {"xx000000000000000000000000000000"}, với xx thay đổi từ 00->13 (giả dụ h t thích check vs 20 plaintext đi ha)
Mỗi lần nhập plaintext vào nhận được số leaked bit, push số đó vào 1 mảng leakbits chẳng hạn
Sau 20 vòng lặp giả sử ta có mảng sau:
leakedbits=[6,7,6,6,7,6,7,....]

Ok h thấy leakedbits[1]=7>N r ha, tức là số leaked bit bị tăng lên, hay + thêm 1 bit á, tức là byte đó ban đầu có LSB[byte[1]]=0, nên h ms đổi làm sao cho lên đc 1 bit là vậy á.

Và khi xor để có mảng leakedbits xong, ta cho vòng lặp i chạy từ 0 tới 255, mỗi vòng i có 1 vòng j chạy từ 0 tới 19, để sinh ra một mảng test cũng lưu các leaked bit. NHƯNG lưu ý, mảng test này chỉ có giá trị là 0 và 1 th nhé, 0 là có LSB=0 và ngược lại. Do đó ta p đổi mảng leakedbits về thành 0 vs 1 dựa trên LSB[byte[1]] ban đầu là 1 hay 0 (đang xét vd ở byte 1 nhé, mk thích đếm byte từ 1 :v) 
Ở trường hợp trên leakedbit có 7>N nên LSB(byte[1]) bđ là bằng 0 nhé, nên chỗ nào trong mảng leakedbits = N thì gán là 0, ngược lại là 1
Còn vd h leakedbits như sau: [6,5,6,6,5,6,5,....], do 5<N=6 nên tức là LSB(byte[1])=1, do đó chỗ nào trong mảng leakedbits = N thì gán là 1, ngược lại là 0 :v

Sửa xong leakedbits r ms chạy for i từ 0 tới 255, trong mỗi vòng for có vòng j từ 0 tới 19, để tạo ra mảng test có 20 phần tử, bằng số phần tử của mảng leakedbits.

Tạo mảng test xong thì so sánh với mảng leakedbits, nếu 2 mảng giống nhau thì byte số 1 chính là số i đó (đổi i từ decimal sang hexa là ra byte số 1 nhé).

Lặp lại liên tục vs 16 byte sẽ ra key cần tìm và gán vào picoCTF{} là xong.

Code lời giải đã có trong file PowerAnalysisWarmup r tự đọc nhé




