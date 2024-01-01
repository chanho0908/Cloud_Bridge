# CloudBridge

## ✔ 프로젝트 소개
> 프렌차이 빵집 중 몇몇 매장의 점주님들은 섭취에 문제가 없지만 특정한 이유로 판매가 불가능한 빵을 
사회 소외 계층들에게 기부해주시고 있습니다. 하지만 현재 이 시스템은 단순히 수기로 작성하고 영수증을
남기는 방식으로 이루어지고 있습니다. 이에 어플리케이션을 제작해 점주님들의 기부를 더욱 활성화 하고 많은 소외 계층들이 도움을
받을 수 있도록 하여 사회적 소외 계층들을 도와주는 다리가 되고자 기획하게 되었습니다.  

### 1️⃣ Language & Libraries
* Kotlin  
* Android Studio

### 2️⃣ 사업자 등록 번호 API
> 목적 : 허위 매장 등록을 방지하기 위해 사업자 등록 번호가 존재하는 매장만 등록이 가능하게 합니다.  
API 출처 : <https://www.data.go.kr/data/15081808/openapi.do>

#### 🎉 사업자 등록 상태조회 API Response
```
사업자등록 상태조회 API Response{
  status_code	string :조회 매칭 수
  request_cnt	integer: 조회 요청 수
  
  data	[
      사업자등록 상태조회 결과{
      b_no : 사업자등록번호
      b_stt	[...]
      b_stt_cd	[...]
      tax_type	01:부가가치세 일반과세자,
                02:부가가치세 간이과세자,
                03:부가가치세 과세특례자,
                04:부가가치세 면세사업자,
                05:수익사업을 영위하지 않는 비영리법인이거나 고유번호가 부여된 단체,국가기관 등,
                06:고유번호가 부여된 단체,
                07:부가가치세 간이과세자(세금계산서 발급사업자),
                * 등록되지 않았거나 삭제된 경우: "국세청에 등록되지 않은 사업자등록번호입니다"
      tax_type_cd	[...]
      end_dt	:폐업일 (YYYYMMDD 포맷)
      utcc_yn	[...]
      tax_type_change_dt	[...]
      invoice_apply_dt	[...]
      rbf_tax_type	[...]
      rbf_tax_type_cd	[...]
      }]
  }]
```

#### 🎉 응답 형태를 바탕으로 데이터를 받아올 data class를 생성합니다.
```
data class CompanyRegistrationNumberState(
    val b_no: String, 
    val b_stt: String, 
    val b_stt_cd: String, 
    val tax_type: String,  
    val tax_type_cd: String,
    val end_dt: String,
    val utcc_yn: String,
    val tax_type_change_dt: String,
    val invoice_apply_dt: String,
    val rbf_tax_type : String,
    val rbf_tax_type_cd: String
)

```
```
data class CprRequestModel (
  val request_cnt: Int,
  val match_cnt: Int,
  val status_code: String,
  val data: List<CompanyRegistrationNumberState>
)
```
#### 🎉 Retrofit interface

```
interface CRNApi {
    @POST("status")
    suspend fun getCRNState(
        @Body requestBody: CrnRequestModel, // 조회할 사업자 등록번호
        @Query("serviceKey") serviceKey:String // API 사용자 인증키
    ): CprRequestModel
}
```
#### 🎉 API 호출 함수
```

class RegistCRNVewModel: ViewModel() {

    private val _state = MutableLiveData<CprRequestModel>()
    val state: LiveData<CprRequestModel>
        get() = _state

    private val CRNApi = CRNRetrofitInstance.getInstance()

    fun getCPRState(bno: String) = viewModelScope.launch(Dispatchers.IO) {
        try {
            val result = CRNApi.getCRNState(CrnRequestModel(arrayOf(bno)), SECRETE_KEY)
            _state.postValue(result)
        }catch (e : Exception){
            Log.d("CPRFragment", "Cause getCPRState: $e")
        }
    }
}
```

### 3️⃣ BackEnd Server  
https://github.com/chanho0908/android_Docker_server

### 4️⃣ 동작 화면
|사업자 등록번호 조회|매장 등록|
|---|---|
|<img src="https://github.com/chanho0908/Cloud_Bridge/assets/84930748/684b1ae4-7536-4244-878d-832b6ba7665c" width="400"/>|<img src="https://github.com/chanho0908/Cloud_Bridge/assets/84930748/9ad3477b-63ed-4e46-b92d-80dcb8b59e6b" width="400"/>|



