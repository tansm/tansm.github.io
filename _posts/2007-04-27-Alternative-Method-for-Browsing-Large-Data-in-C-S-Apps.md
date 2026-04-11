# C/S程序对于大数据量的浏览的另类方法

> 原文：https://www.cnblogs.com/tansm/archive/2007/04/27/729513.html

对于大数据量的用户显示，

B/S

程序几乎清一色的使用分页的方式呈现，一般的，这种方式下用户会查看前几页的数据，当仍然没有找到他所需要的数据时，他会选择重新查询。

在C/S下的程序中，使用分页方式，受到了一些挑战：

一方面，老的滚动条给用户留下了较好的体验，在任何界面下都照搬分页的方式显示，会给客户很不好的印象；

其次，C/S下的应用大多面向商业客户，这些程序涉及到成千上万的数据应用，事实上，大多数的数据量并不大，加上良好的事先条件（例如最近一周的订单），数据量往往不大，但因为分页的缘故，用户哪怕在300条记录的情况下，也要翻动五六下才能看见他的数据；

另外，由于分页的缘故，必然影响浏览时的SQL语句，以适应分页的需求，给程序的编写带来极大的复杂，而且最重要的是分页将本来就繁重的SQL变的更加缓慢，在ERP应用中，权限的条件已经很复杂了，如果再加上分页（我们使用的是双TOP加倒序的方式），SQL执行缓慢无比，比单纯的SQL慢的不是一个级别。

那么如何较好的解决这些问题呢？

首先，从源头开始抓，即默认给用户尽可能少数据量的查询，例如，在产品设计中，使用“我的订单”，或者是“最近一周需要处理的订单”，对于基础档案类的数据，可以尽量在浏览的左边安排一个分类树，以便仅显示此分类的数据。

其次，建立虚拟数据的方式，以便使用表格的方式展现大数据，他的原理是：

建立一个类，实现了IList接口；

表格控件的绑定一般使用IList接口，其中Count和this[int index]是必须实现的方法，Count我们可以通过获取所有Id的列表，得知总数，而实际的数据通过下面的预测性缓存技术获取；

第一次获取查询的所有ID列表，然后根据表格的请求，预测性的获取对应的数据。下面是部分演示代码：

```
获取指定编号的数据
     private Dictionary<object, T> cacheData; //已经获取到的数据

        private int lastIndex = -1; //最后一次访问的索引

        private const int ONE_PAGE_SIZE = 200; //每次读取数据的页面大小

        private T GetDataFromIndex(int index) {

            if ((index <0) || (index>ids.Count -1)){

                throw new IndexOutOfRangeException();

            }

            //获取Id的值

            object id = ids[index];

            //检查Id是否存在缓存中

            T result;

            if (cacheData.TryGetValue(id,out result)) {

                return result;

            }

            //一定会加入当前需要检索的对象

            List<object> searchIds = new List<object>(); //需要寻找的Id列表

            searchIds.Add(id);

            //没有这个数据，根据上次访问的索引确定预测的方向

            int direction = 1 ; //=1 表示向前方向，=-1 表示向后方向

            if (lastIndex <0) {

                direction = 1;

            }

            else {

                direction = ((index >= lastIndex) ? 1 : -1 );

            }

            lastIndex = index;

            //根据方向，将获取其以上的条或者其下面的条

            int newIndex = index + direction;

            while ((newIndex >=0) && (newIndex < ids.Count) && (searchIds.Count <=ONE_PAGE_SIZE)) {

                id = ids[newIndex];

                if (!cacheData.ContainsKey(id)) {

                    searchIds.Add(id);

                }

                newIndex = newIndex + direction;

            }

            //对获取到的数据进行读取

            T[] data = GetData(searchIds);

            //将获取到的数据放入缓存中

            foreach (T item in data) {

                cacheData.Add(GetDataKey(item), item);

            }

            //返回

            return cacheData[ids[index]];

        }

        protected abstract T[] GetData(List<object> ids);

        protected abstract object GetDataKey(T data);
```

通过这种方法，我在SQLServer下的测试，服务器的复杂非常的小，在10万笔记录下，基本上在1秒左右看见数据，滚动数据时基本上没有延迟。

![](/images/2007-04-27-Alternative-Method-for-Browsing-Large-Data-in-C-S-Apps-1.jpg)
当然，当前的实现还有需要改善的地方：

-浏览时，实现读取的Id已经被删除，当滚动此数据后，要使次行空白或自动消失；

-可以考虑首页并不获取所有Id，而是Top 100数据，当用户滚动数据时，才加载所有Id列表，这种设计主要是考虑翻页的概率很低，读取所有Id有些浪费，其次，对于数据量小于100的数据，实际执行了2次SQL，没有必要；

-对于超过10万的记录，获取所有的Id也是很漫长的时间，可以考虑两种策略：1、异步分批下载所有Id，2、告诉用户数据太多了，程序只能处理10万，哈哈。

-用户很无聊，拖动滚动条看遍所有数据，缓存的数据不断增加，应该考虑缓存的最大值，将旧的缓存删除；

-在用户对连续的数据进行频繁的读取时，考虑自动增加每次的读取总数。

此方案给用户的体验是很好的，但可不是完美无缺的，他无法适应以下情况：

-查询的结果是复杂的分组汇总查询，没有可以参考的Id列表，这样情况无法处理；

-必须关闭表格控件的排序、分组和过滤功能，因为这些功能将促使表格访问所有的数据。

下面包含了完整的代码仅供参考，但是你可能下载后无法运行，因为他使用了我们公司自己的ORM，你可以修改成自己的数据结构和读取方式。

```
参考代码
    public abstract class VList<T> : ICollection<T>,IList<T> ,IBindingList{
        private List<object> ids;

        public VList(List<object> idsSource) {
            ids = idsSource;
            cacheData = new Dictionary<object, T>();
        }

        ICollection 成员#region ICollection<T> 成员

        public void Add(T item) {
            throw new Exception("The method or operation is not implemented.");
        }

        public void Clear() {
            throw new Exception("The method or operation is not implemented.");
        }

        public bool Contains(T item) {
            throw new Exception("The method or operation is not implemented.");
        }

        public void CopyTo(T[] array, int arrayIndex) {
            throw new Exception("The method or operation is not implemented.");
        }

        public int Count {
            get { return ids.Count; }
        }

        public bool IsReadOnly {
            get { return true; }
        }

        public bool Remove(T item) {
            throw new Exception("The method or operation is not implemented.");
        }

        #endregion

        IEnumerable 成员#region IEnumerable<T> 成员

        public IEnumerator<T> GetEnumerator() {
            for (int i = 0; i < ids.Count; i++) {
                yield return this[i];
            }
        }

        #endregion

        IEnumerable 成员#region IEnumerable 成员

        System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() {
            for (int i = 0; i < ids.Count; i++) {
                yield return this[i];
            }
        }

        #endregion

        IList 成员#region IList<T> 成员

        public int IndexOf(T item) {
            throw new Exception("The method or operation is not implemented.");
        }

        public void Insert(int index, T item) {
            throw new Exception("The method or operation is not implemented.");
        }

        public void RemoveAt(int index) {
            throw new Exception("The method or operation is not implemented.");
        }

        public T this[int index] {
            get {
                return GetDataFromIndex(index);
            }
            set {
                throw new Exception("The method or operation is not implemented.");
            }
        }

        #endregion

        private Dictionary<object, T> cacheData; //已经获取到的数据的Id
        private int lastIndex = -1; //最后一次访问的索引
        private const int ONE_PAGE_SIZE = 200; //每次读取数据的页面大小

        private T GetDataFromIndex(int index) {
            if ((index <0) || (index>ids.Count -1)){
                throw new IndexOutOfRangeException();
            }

            //获取Id的值
            object id = ids[index];

            //检查Id是否存在缓存中
            T result;
            if (cacheData.TryGetValue(id,out result)) {
                return result;
            }

            //一定会加入当前需要检索的对象
            List<object> searchIds = new List<object>(); //需要寻找的Id列表
            searchIds.Add(id);

            //没有这个数据，根据上次访问的索引确定预测的方向
            int direction = 1 ; //=1 表示向前方向，=-1 表示向后方向
            if (lastIndex <0) {
                direction = 1;
            }
            else {
                direction = ((index >= lastIndex) ? 1 : -1 );
            }
            lastIndex = index;

            //根据方向，将获取其以上的100条或者其下面的100条
            int newIndex = index + direction;
            while ((newIndex >=0) && (newIndex < ids.Count) && (searchIds.Count <=ONE_PAGE_SIZE)) {
                id = ids[newIndex];
                if (!cacheData.ContainsKey(id)) {
                    searchIds.Add(id);
                }
                newIndex = newIndex + direction;
            }

            //对获取到的数据进行读取
            T[] data = GetData(searchIds);

            //将获取到的数据放入缓存中
            foreach (T item in data) {
                cacheData.Add(GetDataKey(item), item);
            }

            //返回
            return cacheData[ids[index]];
        }

        protected abstract T[] GetData(List<object> ids);

        protected abstract object GetDataKey(T data);

        IBindingList 成员#region IBindingList 成员

        public void AddIndex(PropertyDescriptor property) {
            throw new Exception("The method or operation is not implemented.");
        }

        public object AddNew() {
            throw new Exception("The method or operation is not implemented.");
        }

        public bool AllowEdit {
            get { return false; }
        }

        public bool AllowNew {
            get { return false; ; }
        }

        public bool AllowRemove {
            get { return false; ; }
        }

        public void ApplySort(PropertyDescriptor property, ListSortDirection direction) {
            throw new Exception("The method or operation is not implemented.");
        }

        public int Find(PropertyDescriptor property, object key) {
            throw new Exception("The method or operation is not implemented.");
        }

        public bool IsSorted {
            get { return false ; }
        }

        public event ListChangedEventHandler ListChanged;

        public void RemoveIndex(PropertyDescriptor property) {
            throw new Exception("The method or operation is not implemented.");
        }

        public void RemoveSort() {
            throw new Exception("The method or operation is not implemented.");
        }

        public ListSortDirection SortDirection {
            get { return ListSortDirection.Ascending; }
        }

        public PropertyDescriptor SortProperty {
            get { return null; }
        }

        public bool SupportsChangeNotification {
            get { return false; }
        }

        public bool SupportsSearching {
            get { return false ; }
        }

        public bool SupportsSorting {
            get { return false; }
        }

        #endregion

        IList 成员#region IList 成员

        public int Add(object value) {
            throw new Exception("The method or operation is not implemented.");
        }

        public bool Contains(object value) {
            throw new Exception("The method or operation is not implemented.");
        }

        public int IndexOf(object value) {
            throw new Exception("The method or operation is not implemented.");
        }

        public void Insert(int index, object value) {
            throw new Exception("The method or operation is not implemented.");
        }

        public bool IsFixedSize {
            get {return true; }
        }

        public void Remove(object value) {
            throw new Exception("The method or operation is not implemented.");
        }

        object System.Collections.IList.this[int index] {
            get {
                return GetDataFromIndex(index);
            }
            set {
                throw new Exception("The method or operation is not implemented.");
            }
        }

        #endregion

        ICollection 成员#region ICollection 成员

        public void CopyTo(Array array, int index) {
            throw new Exception("The method or operation is not implemented.");
        }

        public bool IsSynchronized {
            get { return false; }
        }

        public object SyncRoot {
            get { return this; }
        }

        #endregion
    }

    public sealed class AccountList : VList<Account> {
        private IDocumentService service;
        public AccountList() : base(GetIds()) {
            Dcms.Common.Torridity.DataSource.SQLServer.SqlServerDriver driver = new Dcms.Common.Torridity.DataSource.SQLServer.SqlServerDriver();
            driver.ConnectionString = Settings.Default.Crm070413ConnectionString;
            service = OrmEntry.CreateDocumentService(OrmEntry.GetDataEntityType(typeof(Account)), driver);
        }

        private static List<object> GetIds() {
            List<object> ids = new List<object>();

            using (SqlConnection con = new SqlConnection(Settings.Default.Crm070413ConnectionString)) {
                con.Open();
                using (SqlCommand cmd = new SqlCommand("Select AccountId FROM Account Order By Code", con)) {
                    using (SqlDataReader reader = cmd.ExecuteReader(CommandBehavior.SingleResult)) {
                        while (reader.Read()) {
                            ids.Add(reader.GetString(0));
                        }
                        reader.Close();
                    }
                }
                con.Close();
            }
            return ids;
        }
        protected override Account[] GetData(List<object> ids) {
            object[] result = service.Read(ids.ToArray(), false);
            Account[] r2 = new Account[result.Length];
            Array.Copy(result, r2, result.Length);

            Console.WriteLine("Read Data");

            return r2;
        }

        protected override object GetDataKey(Account data) {
            return data.AccountId;
        }
    }
```

和表格控件绑定的代码：

```
        private void button2_Click(object sender, EventArgs e) {
            list = new AccountList();
            gridControl1.DataSource = list;
        }
```
