# 開發端筆記

歡迎留言或編輯，以完善這份範本

## Tips & Tricks

- 無法寫入路徑或檔案時：資料夾要加一個叫"AUTH"的權限角色，給他要用的權限

## 方法範本

### C#

後端Method/一般方法模板

```csharp=
/*
目的：存檔檢查某物件
註記：2025-01-06 Nicholas created
觸發：
作法：
輸入：
輸出：
*/
// 給予超級使用者權限
// Aras.Server.Security.Identity plmIdentity = Aras.Server.Security.Identity.GetByName("Super User");
// bool PermissionWasSet = Aras.Server.Security.Permissions.GrantIdentity(plmIdentity);
// 簽審用權限
// Aras.Server.Security.Identity plmIdentityAras = Aras.Server.Security.Identity.GetByName("Aras PLM"); // for promote
// bool PermissionArasWasSet = Aras.Server.Security.Permissions.GrantIdentity(plmIdentityAras);

Innovator inn = this.getInnovator();
string strDatabaseName = inn.getConnection().GetDatabaseName();
DateTime dateTimeNow = DateTime.Now;
string strMethodNamePart = InMethodName;
string strMethodName =  string.Format("[{0}][{1}]{2}[{3}]", strDatabaseName, dateTimeNow.ToString("yyyy-MM-dd_hh-mm-ss-fff"), strMethodNamePart, inn.getNewID() );
// 當天瘋狂 debug 開關
strMethodName =  string.Format("[{0}][{1}]{2}", strDatabaseName, dateTimeNow.ToString("yyyy-MM-dd"), strMethodNamePart );
bool is_testing = true;
bool is_debugging = true;
string strMethodPrupose = "";

Innosoft.InnovatorHelper _InnH = new Innosoft.InnovatorHelper(inn);
string aml = "";
string sql = "";
// string strUserId = inn.getUserID();
// string strIdentityId = inn.getUserAliases();
// string strIdentityId = CCO.Identity.GetUserAliases(strUserId);// aras 2023
// string idenIds = Aras.Server.Security.Permissions.GetIdentitiesList(CCO.DB.InnDatabase, strUserId);

_InnH.AddLog(strMethodName, "MethodSteps");
// CCO.Utilities.WriteDebug(strMethodName, InGCL(DateTime.Now.ToString("hh:mm:ss.fff") + "\n" +"this\n" + this.ToString() ));
Item itmThis = this;
// CCO.Utilities.WriteDebug(strMethodName, InGCL("itmThis = \n" + FormatXml(itmThis.dom.InnerXml)));
CCO.Utilities.WriteDebug(strMethodName,InGCL("in"));

#region 嘗試取得物件標籤，用於報錯

// // 設定要取得的物件類型名稱
// string[] arrItemtypeName = { "in_proposal", "in_proposal_team" };

// // 抓不到標籤將會帶 arrItemtypeName 的值，且物件不存在的話會在值的前方加上字串 "不存在的物件類型："
// Dictionary<string, string> dictItemtypeLabel = getItemtypeLabelDict(arrItemtypeName);

#endregion

try
{
    #region input

    // init stuff

    #endregion

    #region local variable

    // define stuff
    string strErrorMsg = "";

    #endregion

    #region process

    // do stuff

    #endregion

    // end of try
}
catch (Exception ex)
{
    string strExceptionMsg = "";
    if (strMethodPrupose == "") strMethodPrupose = strMethodName;
    System.Diagnostics.StackTrace trace = new System.Diagnostics.StackTrace(ex, true);
    int int_error_line = trace.GetFrame(0).GetFileLineNumber() - current_line_number_offset ;
    strExceptionMsg += strMethodPrupose + "，執行失敗，原因為：" + ex.Message;
    CCO.Utilities.WriteDebug(strMethodName, "At Line [" + int_error_line.ToString() + "]\r\n" + strExceptionMsg );
    // 替換物件類型名稱為物件類型標籤
    foreach ( var strItemtypeName in arrItemtypeName ){
        strExceptionMsg = System.Text.RegularExpressions.Regex.Replace(strExceptionMsg, @"\[" + strItemtypeName + @"\[", "[" + dictItemtypeLabel[strItemtypeName] + "]", RegexOptions.IgnoreCase);
        // strExceptionMsg = strExceptionMsg.Replace( "[" + strItemtypeName + "]", "[" + dictItemtypeLabel[strItemtypeName] + "]" );
    }
    throw new Exception(strExceptionMsg);
}
// CCO.Utilities.WriteDebug(strMethodName, InGCL("itmThis = \n" + FormatXml(itmThis.dom.InnerXml)));
// if (PermissionWasSet) Aras.Server.Security.Permissions.RevokeIdentity(plmIdentity);
// if (PermissionArasWasSet) Aras.Server.Security.Permissions.RevokeIdentity(plmIdentityAras);
return itmThis;
}

// 使用 sql 取得所有想取得的物件標籤
private Dictionary<string, string> getItemtypeLabelDict(string[] arrItemtypeName, Dictionary<string, string> dictItemtypeLabel = null){
    Innovator inn = this.getInnovator();
    // 字典鍵值忽略大小寫
    if (dictItemtypeLabel == null) {
        dictItemtypeLabel = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
    } else {
        dictItemtypeLabel = new Dictionary<string, string>(dictItemtypeLabel, StringComparer.OrdinalIgnoreCase);
    }
    // 沒有需要查的物件類型就返回空字典
    if ( arrItemtypeName.Length == 0 ){
        return dictItemtypeLabel;
    }
    // 組合 sql 查詢條件
    string sql = @"
    SELECT RT.[LABEL_ZT] 'rt_label'
    , I.[LABEL_ZT] 'label'
    , I.[NAME] 'name'
    FROM [ITEMTYPE] I
    LEFT JOIN [RelationshipType] RT ON I.[id] = RT.[RELATIONSHIP_ID]
    WHERE";
    string sqlTemplate = @" I.[NAME] = '{#itemtypeName}'";
    string sqlWhere = "";
    for ( int i = 0 ; i < arrItemtypeName.Length ; i++ ){
        if ( sqlWhere != "" ){
            sqlWhere += @"
            OR";
        }
        sqlWhere += sqlTemplate.Replace("{#itemtypeName}", arrItemtypeName[i]);
    }
    sql += sqlWhere;
    // 執行查詢，儲存關聯類型標籤/物件類型標籤
    try {
        Item tempS = inn.applySQL(sql);
        if ( tempS.isError() || tempS.getResult() == "" ){
            // 取得物件類型的中文標籤失敗
        } else {
            for ( int i = 0 ; i < tempS.getItemCount() ; i++ ){
                Item temp = tempS.getItemByIndex(i);
                string name = temp.getProperty("name", "");
                string label = temp.getProperty("label", "");
                string rt_label = temp.getProperty("rt_label", "");
                if ( rt_label != "" && label != "" ){
                    dictItemtypeLabel[name] = rt_label + "/" + label;
                } else if( label != "" ){
                    dictItemtypeLabel[name] = label;
                } else if ( rt_label != "" ){
                    dictItemtypeLabel[name] = rt_label;
                } else if ( name != "" ){
                    dictItemtypeLabel[name] = name;
                }
            }
        }
    } catch(Exception ex){
        // 抓不到就算了 不到很重要
    }
    // 處理沒有抓到資料的輸入值
    for ( int i = 0 ; i < arrItemtypeName.Length ; i++ ){
        if ( !dictItemtypeLabel.ContainsKey(arrItemtypeName[i]) ){
            dictItemtypeLabel[arrItemtypeName[i]] = "不存在的物件類型：" + arrItemtypeName[i];
        }
    }
    return dictItemtypeLabel;
}

// 取得方法名稱
private static string MethodName = "$(pkgname)".Substring(4,"$(pkgname)".LastIndexOf('_', "$(pkgname)".LastIndexOf('_')-1)-4);

// 取得事件名稱
private static string EventName = new System.Text.StringBuilder("$(EventDataClass)").Replace(@"Aras.Server.Core.", "").Replace(@"EventArgs", "").ToString();

// 請隨時改為當前方法範本的 line_number_offset
private const int get_current_line_number_offset = 36;

// 只回傳行號的版本，使用範例： CCO.Utilities.WriteDebug(strMethodName,GCL());
public static string GCL([System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return (lineNumber - get_current_line_number_offset).ToString();
}

// 有額外參數，並回傳字串的版本，使用範例： CCO.Utilities.WriteDebug(strMethodName, GCL( "this: " + this.ToString() ));
public static string GCL(string extraInfo, [System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return "Line Number: " + "[" + (lineNumber - get_current_line_number_offset).ToString() + "]" + "\r\n" + extraInfo + "";
}

public void fin(){
```

更新物件內容的METHOD範例

```csharp=
/*
目的：存檔檢查某物件
日期：2024-09-00 0000 YourNameHere_這邊永遠都要是最後更新時間及人員
觸發：
作法：
輸入：
輸出：
註記：2024-09-00 YourNameHere created
*/
// 給予超級使用者權限
// Aras.Server.Security.Identity plmIdentity = Aras.Server.Security.Identity.GetByName("Super User");
// bool PermissionWasSet = Aras.Server.Security.Permissions.GrantIdentity(plmIdentity);
// 簽審用權限
// Aras.Server.Security.Identity plmIdentityAras = Aras.Server.Security.Identity.GetByName("Aras PLM"); // for promote
// bool PermissionArasWasSet = Aras.Server.Security.Permissions.GrantIdentity(plmIdentityAras);

Innovator inn = this.getInnovator();
string strDatabaseName = inn.getConnection().GetDatabaseName();
DateTime dateTimeNow = DateTime.Now;
string strMethodNamePart = "In_method_name";
string strMethodName =  string.Format("[{1}][{2}]{0}", strMethodNamePart, strDatabaseName, dateTimeNow.ToString("yyyy-MM-dd_hh-mm-ss"));
string strMethodPrupose = "存檔檢查某物件";

Innosoft.InnovatorHelper _InnH = new Innosoft.InnovatorHelper(inn);
string aml = "";
string sql = "";
// string strUserId = inn.getUserID();
// string strIdentityId = inn.getUserAliases();
// string idenIds = Aras.Server.Security.Permissions.GetIdentitiesList(CCO.DB.InnDatabase, strUserId);

_InnH.AddLog(strMethodName, "MethodSteps");
// CCO.Utilities.WriteDebug(strMethodName, DateTime.Now.ToString("hh:mm:ss.fff") + "\n" +"this\n" + this.ToString() );
Item itmThis = this;
// CCO.Utilities.WriteDebug(strMethodName, "itmThis = \n" + itmThis.dom.InnerXml);

try
{
    
    // 看 dot .net framwork 版本
    try{
        string strR = "";
        strR = Environment.Version.ToString();
        return inn.newResult(strR);
    } catch (Exception ex){
        // 娃
    }
    
    /*input*/
    // string in_date_s = itmThis.getProperty("created_on", nowStr);

    /*local variable*/
    Dictionary<string, string> updateColumnSql = new Dictionary<string, string>();
    Dictionary<string, string> updateColumnSqlSub = new Dictionary<string, string>();
    Dictionary<string, string> updateColumnApi = new Dictionary<string, string>();
    Item itmUpdateItem = null;
    bool hasUpdateColumnApi = false;
    bool hasUpdateColumnSql = false;
    // string nowStr = dateTimeNow.ToString("yyyy-MM-ddTHH:mm:ss");

    /*process*/
    // updateColumnSql["in_date_s"] = (DateTime.Parse(in_date_s).AddHours(-8)).ToString("yyyy-MM-ddTHH:mm:ss");
    
    if ( updateColumnApi.Count > 0 ){
        hasUpdateColumnApi = true;
        // 組合單頭更新 aras api
        itmUpdateItem = inn.newItem(itmThis.getType(), "edit");
        itmUpdateItem.setAttribute("where", "id='" + itmThis.getID() + "'");
        updateColumnSql.Keys.ToList().ForEach(k =>
            itmUpdateItem.setProperty(k, updateColumnSql[k])
        );
        CCO.Utilities.WriteDebug(strMethodName, "itmUpdateItem = \n" + itmUpdateItem.ToString());
        
    }
    if ( updateColumnSql.Count > 0 || updateColumnSqlSub.Count > 0) {
        hasUpdateColumnSql = true;
        // 組合單頭更新 sql
        char[] charsToTrim = { ',', ' '};
        sql = "update [" + itmThis.getType() + "] set ";
        updateColumnSql.Keys.ToList().ForEach(k => 
            sql += k + "='" + updateColumnSql[k].Replace("'", "''") + "', "
        );
        updateColumnSqlSub.Keys.ToList().ForEach(k => 
            sql += k + "=" + updateColumnSqlSub[k] + ", "
        );
        sql = sql.Trim(charsToTrim);
        sql += " where id='" + itmThis.getID() + "'";
        CCO.Utilities.WriteDebug(strMethodName, "sql = \n" + sql);
        
    }
    
    if ( hasUpdateColumnApi || hasUpdateColumnSql  ){
        CCO.Utilities.WriteDebug(strMethodName, "執行更新");
        
        Item itmReturn = inn.newItem();
        
        
        if ( hasUpdateColumnApi ){
            try {
                itmReturn = itmUpdateItem.apply();
                if ( itmReturn.isError() ){
                    throw new Exception( itmReturn.getResult().ToString() );
                }
            } catch ( Exception ex ){
                throw new Exception("觸發事件的更新失敗:\n" + ex.Message );
            }
        }
        
        
        if ( hasUpdateColumnSql ){
            try{
                itmReturn = inn.applySQL(sql);
                if ( itmReturn.isError() ){
                    throw new Exception( itmReturn.getResult().ToString() );
                }
            } catch ( Exception ex ){
                throw new Exception( "不觸發事件的更新失敗:\n" + ex.Message );
            }
        }
        
    }
    else {
        CCO.Utilities.WriteDebug(strMethodName, "未執行更新，因為沒有需要更新的欄位");
    }
    // end of try
}
catch (Exception ex)
{
    if (strMethodPrupose == "") strMethodPrupose = strMethodName;
    CCO.Utilities.WriteDebug(strMethodName, strMethodPrupose + "執行失敗，原因為：" + ex.Message);
    throw new Exception(strMethodPrupose + "執行失敗，原因為：" + ex.Message);
}
// CCO.Utilities.WriteDebug(strMethodName, "itmThis = \n" + itmThis.dom.InnerXml);
// if (PermissionWasSet) Aras.Server.Security.Permissions.RevokeIdentity(plmIdentity);
// if (PermissionArasWasSet) Aras.Server.Security.Permissions.RevokeIdentity(plmIdentityAras);
return itmThis;
```

額外過濾 get 結果 並指定權限方法

```csharp=
/*
目的：案場 對非管理員 只出現 實際報工工地 與 主專案
觸發：onBeforeGet
作法：
輸入：
輸出：
日期：created by Louis
2024-05-13 Nicholas 修改 讓高權限使用者不受限制
*/

Innovator inn = this.getInnovator();
string strDatabaseName = inn.getConnection().GetDatabaseName();
string strMethodName =  string.Format("[{0}][{1}]{2}", strDatabaseName, DateTime.Now.ToString("yyyy-MM-dd_hh-mm-ss"), "In_warehouse_onbeforeget");
string strMethodPrupose = "案場只出現實際報工工地與主專案";

Innosoft.InnovatorHelper _InnH = new Innosoft.InnovatorHelper(inn);
string aml = "";
string sql = "";
string strUserId = inn.getUserID();
//string strIdentityId = inn.getUserAliases();
string idenIds = Aras.Server.Security.Permissions.GetIdentitiesList(CCO.DB.InnDatabase, strUserId);

_InnH.AddLog(strMethodName, "MethodSteps");
Item itmThis = this;

try
{
    /*input*/

    /*local variable*/
    bool isPassIdentityCheck = false;
    // Administrator 角色的 config_id
    string AdministratorsIdentityConfigId = "2618D6F5A90949BAA7E920D1B04C7EE1";

    /*process*/
    // 取得 Administrators 角色
    Item itmIdentity_Administrators = inn.newItem("Identity", "get");
    // 搜尋條件
    itmIdentity_Administrators.setProperty("config_id", AdministratorsIdentityConfigId);
    // 節省效能
    itmIdentity_Administrators.setAttribute("select", "config_id,id");
    itmIdentity_Administrators = itmIdentity_Administrators.apply();
    
    string identityId_administrator = "";
    if ( !itmIdentity_Administrators.isError() && !itmIdentity_Administrators.isEmpty() ){
        identityId_administrator = itmIdentity_Administrators.getID();
    }
    
    if ( identityId_administrator != "" ){
        // 檢查 使用者權限 是否包含 特定使用權
        isPassIdentityCheck = idenIds.Contains(identityId_administrator);
    }
    // 如果通過 就不新增過濾 get 的條件
    if ( !isPassIdentityCheck ){
        this.setProperty("in_class","'real','top'");
        this.setPropertyAttribute("in_class","condition","in");
        this.setAttribute("orderBy","created_on desc");
    }
    // end of try
}
catch (Exception ex)
{
    if (strMethodPrupose == "") strMethodPrupose = strMethodName;
    throw new Exception(strMethodPrupose + "執行失敗，原因為：" + ex.Message);
}
//CCO.Utilities.WriteDebug(strMethodName, "this = \n" + itmThis.dom.InnerXml);
return this;

// /*
// Nicholas 舊版備份
// 位置:onBeforeGet
// 只出現 實際報工工地 與 主專案
// */
// this.setProperty("in_class","'real','top'");
// this.setPropertyAttribute("in_class","condition","in");
// return this;

// /*
// 日期: 2024-06-06 Nicholas
// 位置: onBeforeGet
// 以建立日期降序排序
// */
// this.setAttribute("orderBy","created_on desc");
// return this;
```

### JavaScript

#### 搜尋窗限制方法範本

https://app3.innosoft.com.tw/plm/Client/X-salt=26_11.0.0.6549-X/scripts/Innovator.aspx?db=Innovator

來自誠逸

在關係類型中，要卡的物件屬性的屬性值打開，有一個關係類型叫做event，在裡面加入javascript方法，觸發事件是OnSearchDialog：
```javascript=
var inn = this.getInnovator();
var sql = "";
var strIDList = "";
var idArray = new Array();
var retObj = new Object();
var source_id = thisItem.getProperty("id","");
debugger;

var sql = "select * from in_paylist_vpay where source_id = '"+source_id+"' and isnull(in_vendor_payment,'') != '';";
var itmPaylistVpays = inn.applyMethod("In_ApplySQL","<sql>"+sql+"</sql>");
//alert(sql);

for(var i=0;i<itmPaylistVpays.getItemCount();i++)
{
    var itmPaylistVpay = itmPaylistVpays.getItemByIndex(i);
    idArray[i] = itmPaylistVpay.getProperty("in_vendor_payment","");
}
if (idArray.length > 0)
{
    strIDList = idArray.join(",");
    inArgs.QryItem.item.setAttribute("idlist", strIDList);
    return("");
}
else{
    alert(this.getInnovator().applyMethod("In_Translate","<text>查無 同批次廠商請款審核單的相關廠商請款單</text>").getResult());
    retObj["item_number"] = {filterValue: "N/A", isFilterFixed: true};
    return retObj;
}

/*if(proposal_id=="")
{
    alert(inn.applyMethod("In_Translate","<text>請先選擇案件登記表</text>").getResult());
    retObj["item_number"] = {filterValue:'N/A',isFilterFixed:true};
    return retObj;
}
else
{
    //2022-06-07 改判斷所有相關的立案單
    //2022-06-15 改丟入後端搜尋
    //sql = "SELECT id FROM In_Cadastral WHERE in_proposal='" + proposal_id + "'";
    //sql = "SELECT id FROM In_Cadastral WHERE in_proposal_t='" + proposal_id + "' OR in_proposal_s='" + proposal_id + "' OR in_proposal_e ='" + proposal_id + "' OR in_proposal_o='" + proposal_id + "' OR in_proposal_d='" + proposal_id + "'";
    var itmCadastralcs = inn.applyMethod("In_SetClDateSelCad_S","<proposal_id>"+proposal_id+"</proposal_id>");
        
    for(var i=0;i<itmCadastralcs.getItemCount();i++)
    {
        var itmCadastralc = itmCadastralcs.getItemByIndex(i);
        idArray[i] = itmCadastralc.getID();
    }
    if (idArray.length > 0)
    {
        strIDList = idArray.join(",");
        inArgs.QryItem.item.setAttribute("idlist", strIDList);
        return("");
    }
    else{
        alert(this.getInnovator().applyMethod("In_Translate","<text>查無 同立案單下有相關的產權</text>").getResult());
        retObj["item_number"] = {filterValue: "N/A", isFilterFixed: true};
        return retObj;
    }
}*/
```

<!--
#### aras API 使用 Property 設定排序條件
```C#=
using Aras.IOM;

class Program
{
    static void Main(string[] args)
    {
        // 連接到 Aras Innovator
        Innovator inn = new Innovator();

        // 創建 AML 查詢
        Item queryItem = inn.newItem("Item", "get");
        queryItem.setAttribute("type", "ItemType");
        
        // 添加排序條件
        Item orderByItem = queryItem.createRelationship("orderBy");
        Item propertyItem = orderByItem.createItem("property");
        propertyItem.setProperty("keyedName", "你的屬性名稱");
        propertyItem.setProperty("direction", "asc"); // 或者 "desc" 表示降序

        // 執行查詢
        Item resultItem = queryItem.apply();
        if (resultItem.isError())
        {
            // 處理錯誤
            string errorMessage = resultItem.getErrorString();
            Console.WriteLine("Error: " + errorMessage);
        }
        else
        {
            // 處理查詢結果
            // resultItem 包含查詢結果
        }
    }
}
```
-->

#### 執行動作後 自動重新載入新資料

來自 士電 In_PostOcr_Document_Client

```javascript=
debugger;
/*
 OCR建立按鈕回饋事件 2023.05.31 created by panda 
*/
var tmpThisItem = typeof(parent.document.thisItem) == "object" ? parent.document.thisItem : parent.thisItem;
var inn = tmpThisItem.getInnovator();
tmpThisItem.setProperty("model_name","prebuilt-read");
var result = tmpThisItem.apply("In_PostOcr_Document");

// 顯示執行結果
if(result.isError()){
    top.aras.AlertError(result.getErrorString());
}else{
    top.aras.AlertWarning("更新完成");
}

// 重新載入自身物件，
var itmData = inn.newItem("Document","get");
itmData.setProperty("id",tmpThisItem.getProperty("id",""));
itmData = itmData.apply();

top.aras.itemsCache.addItem(itmData.node);
top.aras.uiReShowItemEx(itmData.node.getAttribute("id"),itmData.node, "tab view");
top.aras.uiShowItemEx(itmData.node, "tab view");
```

找出欄位精度及小數位數

```sql=
SELECT 
    COLUMN_NAME, 
    DATA_TYPE, 
    NUMERIC_PRECISION, 
    NUMERIC_SCALE
FROM 
    INFORMATION_SCHEMA.COLUMNS
WHERE 
    TABLE_NAME = 'IN_CADASTRAL'
    AND COLUMN_NAME = 'IN_AREA_M';
```

找出資料庫上次被存取

```sql=
SELECT 
    object_name(OBJECT_ID) AS TableName,
    last_user_update,
    last_user_seek,
    last_user_scan,
    last_user_lookup
FROM 
    sys.dm_db_index_usage_stats
WHERE 
    database_id = DB_ID('PLM23');
```

# AML/XML 範本

## 取得多語系

取得值

```XML=
<Item type="Property" language='en,zt,zc' action="get" select='label'>
  <name condition="eq">in_weight_finished_time</name>
</Item>
```

得到的結果

```XML=
<SOAP-ENV:Envelope xmlns:i18n="http://www.aras.com/I18N" xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <SOAP-ENV:Body>
    <Result>
      <Item type="Property" typeId="26D7CD4E033242148E2724D3D054B4D3" id="1526E45C3301416E82A6112E2757DE08">
        <id keyed_name="in_weight_finished_time" type="Property">1526E45C3301416E82A6112E2757DE08</id>
        <label is_null="1" xml:lang="en" />
        <i18n:label is_null="1" xml:lang="zc" />
        <i18n:label xml:lang="zt">in_weight_finished_time</i18n:label>
        <i18n:label is_null="1" xml:lang="en" />
      </Item>
    </Result>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

做成以下更新用格式，執行AML

```XML=
<AML>
  <Item type="Property" typeId="26D7CD4E033242148E2724D3D054B4D3" action="edit" where="id='1526E45C3301416E82A6112E2757DE08'">
    <id keyed_name="in_weight_finished_time" type="Property">1526E45C3301416E82A6112E2757DE08</id>
    <label is_null="1" xml:lang="en" />
    <i18n:label xmlns:i18n="http://www.aras.com/I18N" is_null="1" xml:lang="zc" />
    <i18n:label xmlns:i18n="http://www.aras.com/I18N" xml:lang="zt">in_weight_finished_time</i18n:label>
    <i18n:label xmlns:i18n="http://www.aras.com/I18N" is_null="1" xml:lang="en" />
  </Item>
</AML>
```

# SQL

## 將某一筆資料，暫存到暫存資料庫，並產生匯入用指令

```sql=
SELECT * INTO AAA_In_Meeting_ShortUrl_01
FROM innovator.In_Meeting_ShortUrl
WHERE id = '14CC118C282E4B11985F30D750566E6E'
```

再對著「資料庫」點右鍵 > 工作 > 產生指令碼

在「選擇物件」頁面點 選取特定的資料庫物件 > 選資料表 > 剛剛建立的暫存資料表 打勾

在「設定指令碼編寫選項」 選取儲存為指令碼 > 選單一指令檔

在「設定指令碼編寫選項」 按進階 > 一般 > 要編寫指令碼的資料類型 > 僅限資料

一路按到完成 開啟

將暫存的資料表 改為目的地真正的資料表名稱 例如：

`[dbo].[AAA_In_Meeting_ShortUrl_01]` 改為 `innovator.[In_Meeting_ShortUrl]`

執行 sql

# 納骨塔

食之無味 棄之可惜

```csharp

#region 取得當前行號字串方法，已整合至 Method Config

// 請將 36 改為當前方法範本的 line_number_offset
public static int current_line_number_offset{
    return 36;
}
// 精簡版本，使用範例： CCO.Utilities.WriteDebug(strMethodName,InGCL());
public static string InGCL([System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return GetCallerLineNumber(lineNumber);
}
public static string InGCL(string extraInfo, [System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return GetCallerLineNumber(extraInfo, lineNumber);
}
// 只回傳行號的版本，使用範例： CCO.Utilities.WriteDebug(strMethodName,GetCallerLineNumber());
public static string GetCallerLineNumber([System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return (lineNumber - current_line_number_offset).ToString();
}
// 有額外參數，並回傳字串的版本，使用範例： CCO.Utilities.WriteDebug(strMethodName, GetCallerLineNumber( "this: " + this.ToString() ) );
public static string GetCallerLineNumber(string extraInfo, [System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return "Line Number: " + "[" + (lineNumber - current_line_number_offset).ToString() + "]" + "\r\n" + extraInfo + "";
}

// regex 尋找
CCO\.Utilities\.WriteDebug\s*\(\s*strMethodName\s*,\s*"\s*\d+\s*"\s*\)\s*;
// 替換為
CCO.Utilities.WriteDebug(strMethodName, InGCL() );

#endregion

#region 取得方法名稱及取得事件名稱，已整合至 Method Config

// 取得方法名稱，原型概念
public static string GetMethodName(){
    string methodNamePattern = @"(?<=at PKG_).*(?=_.*_.*\.ItemMethod\.methodCode\([^\)]*\))";
    Match methodNameMatch = Regex.Match(new System.Diagnostics.StackTrace().ToString(), methodNamePattern);
    if (methodNameMatch.Success){
        return methodNameMatch.Value;
    }
    return "";
}

// 取得事件名稱，原型概念
public static string GetEventName(){
    string methodEventPattern = @"(?<=at PKG_.*\.ItemMethod\.methodCode\()([^)]*)(?=\s*EventArgs eventData\))";
    Match methodEventMatch = Regex.Match(new System.Diagnostics.StackTrace().ToString(), methodEventPattern);
    if (methodEventMatch.Success){
        return methodEventMatch.Value;
    }
    return "";
}

// 取得方法名稱，優化效能後
private static string MethodName = "$(pkgname)".Substring(4,"$(pkgname)".LastIndexOf('_', "$(pkgname)".LastIndexOf('_')-1)-4);

// 取得事件名稱，優化效能後
private static string EventName = new System.Text.StringBuilder("$(EventDataClass)").Replace(@"Aras.Server.Core.", "").Replace(@"EventArgs", "").ToString();

#endregion

#region xml 字串排版

// xml 字串排版
public string FormatXml(string xml){
    try{
        // 使用 XmlDocument 來載入 XML 字串
        System.Xml.XmlDocument doc = new System.Xml.XmlDocument();
        doc.LoadXml(xml);

        // 設定格式化選項
        System.Text.StringBuilder sb = new System.Text.StringBuilder();
        System.Xml.XmlTextWriter writer = new System.Xml.XmlTextWriter(new System.IO.StringWriter(sb));
        writer.Formatting = System.Xml.Formatting.Indented;
        
        // 將 XML 輸出到 StringBuilder 中，實現格式化
        doc.Save(writer);

        return sb.ToString();
    }
    catch (Exception ex){
        // 若格式化過程中發生錯誤，直接返回原始的 XML 字串
        return xml;
    }
}

#endregion

```

各環境都兼容版本，可以跟改過 method-config 的方法並用不衝突

```csharp
// 取得方法名稱
private static string MethodName = "$(pkgname)".Substring(4,"$(pkgname)".LastIndexOf('_', "$(pkgname)".LastIndexOf('_')-1)-4);

// 取得事件名稱
private static string EventName = new System.Text.StringBuilder("$(EventDataClass)").Replace(@"Aras.Server.Core.", "").Replace(@"EventArgs", "").ToString();

// 請隨時改為當前方法範本的 line_number_offset
private const int get_current_line_number_offset = 36;

// 只回傳行號的版本，使用範例： CCO.Utilities.WriteDebug(strMethodName,GCL());
public static string GCL([System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return (lineNumber - get_current_line_number_offset).ToString();
}

// 有額外參數，並回傳字串的版本，使用範例： CCO.Utilities.WriteDebug(strMethodName, GCL( "this: " + this.ToString() ));
public static string GCL(string extraInfo, [System.Runtime.CompilerServices.CallerLineNumber] int lineNumber = 0){
    return "Line Number: " + "[" + (lineNumber - get_current_line_number_offset).ToString() + "]" + "\r\n" + extraInfo + "";
}

public void fin(){
```