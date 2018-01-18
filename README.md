# RealmSqlite
Realm操作数据库工具类 【具体请查看工具类】

      在项目build下加入：
       dependencies {
            classpath "io.realm:realm-gradle-plugin:2.2.1"
      }
     在App build下加入
     apply plugin: 'realm-android'

# 初始化
      RealmSqlite.getRealmSqlite().initRealm(this);
# 增 删 改 查
### 增
       /**
         * 数据库增加数据，没有主键@PrimaryKey  add
         * */
        public <D extends RealmModel> void addRealm(final D d){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    mRealm.copyToRealm(d);
                }
            });

        }
        /**
         * 数据库增加数据，有主键@PrimaryKey  add
         * */
        public <D extends RealmModel> void addRealmKey(final D d){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    mRealm.copyToRealmOrUpdate(d);
                }
            });

        }
        /**
         * 数据库增加数据,通过回调方式  add
         * */
        public <D extends RealmModel> void add(D d,OnChangeRelmListener listener){
            mRealm.beginTransaction();//开启事务
            D bean = (D) mRealm.createObject(d.getClass());
            listener.change(bean);
            mRealm.commitTransaction();//提交事务
        }

        /**
         * 在UI和后台线程同时开启创建write的事务，可能会导致ANR错误。为了避免该问题，可以使用executeTransactionAsync来实现
         * */
        public <D extends RealmModel> void addAsync(final D d,final OnChangeRelmListener listener){
            RealmAsyncTask transaction = mRealm.executeTransactionAsync(new Realm.Transaction() {
                @Override
                public void execute(Realm realm) {
                    D bean = (D) realm.createObject(d.getClass());
                    listener.change(bean);
                }
            });
        }

        /**
         * 添加数据，设置监听回调，检测是否添加成功
         * */

        private  RealmAsyncTask transaction;
        public <D extends RealmModel> void addAsyncState(final D d,final OnChangeRelmListener listener){
             transaction =  mRealm.executeTransactionAsync(new Realm.Transaction() {
                @Override
                public void execute(Realm realm) {

                }
            }, new Realm.Transaction.OnSuccess() {
                @Override
                public void onSuccess() {
                    //成功回调
                    D bean = (D) mRealm.createObject(d.getClass());
                    listener.change(bean);
                }
            }, new Realm.Transaction.OnError() {
                @Override
                public void onError(Throwable error) {
                    //失败回调
                }
            });
        }
        /**
         * 如果当Acitivity或Fragment被销毁时，在OnSuccess或OnError中执行UI操作，将导致程序奔溃 。用RealmAsyncTask .cancel();可以取消事务 在onStop中调用，避免crash
         * */
        public void onStopAsync () {
            if (transaction != null && !transaction.isCancelled()) {
                transaction.cancel();
            }
        }

        /**
         * 使用Json字符串插入数据 没有主键
         * */
        public <D extends RealmModel> void addJson(final D d,final String json,final OnChangeRelmListener listener){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm realm) {
                    mRealm.createObjectFromJson(d.getClass(), json);
                }
            });
        }
        /**
         * 使用Json字符串插入数据  有主键
         * */
        public <D extends RealmModel> void addJsonKey(final D d,final String json,final OnChangeRelmListener listener){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm realm) {
                    mRealm.createOrUpdateObjectFromJson(d.getClass(), json);
                }
            });
        }
### 删
          /**
         * 删除完
         * */
        public <D extends RealmModel> void delAll(final D d){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    mRealm.delete(d.getClass());
                }
            });
        }

        public <D extends RealmModel> void delRealm(final D d){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    RealmResults<D> userList = (RealmResults<D>) mRealm.where(d.getClass()).findAll();
                    userList.deleteAllFromRealm();
                }
            });
        }

        /**
         * 删除某一个
         * */
        public <D extends RealmModel> void delRealmPosition(final D d,final int position){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    RealmResults<D> userList = (RealmResults<D>) mRealm.where(d.getClass()).findAll();
                    userList.deleteFromRealm(position);
                }
            });
        }
  ### 改
       /**
         * 更改第一个
         * */
        public <D extends RealmModel> void changeFirst(final D d,final OnChangeRelmListener listener){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    //先查找后得到User对象
                    D bean = (D) mRealm.where(d.getClass()).findFirst();
                    listener.change(bean);
                }
            });
        }
        /**
         * 更改数据库数据 change
         * */
        public <D extends RealmModel> void changeRealm(final D d,final OnChangeRelmListener listener){
            mRealm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm r) {
                    try {
                        RealmResults<D> list = (RealmResults<D>) mRealm.where(d.getClass()).findAll();
                        listener.change(list.get(list.size()-1));
                    }catch (Exception e){
                        e.printStackTrace();
                    }

                }
            });
        }
  
### 查
       /**
         * 查找第一条数据
         * */
        public <D extends RealmModel> D findFirst(D d){
            D bean = (D) mRealm.where(d.getClass()).findFirst();
            return bean;
        }
        /**
         *根据条件进行获取
         * name为value的所有数据
         * */
        public <D extends RealmModel> D findRealmSelected(D d,String name,String value){
            RealmResults<D> userList = (RealmResults<D>)mRealm.where(d.getClass())
                    .equalTo(name, value).findAll();
            return userList.first();
        }
        /**
         * 查找数据
         * */
        public <D extends RealmModel> D findAll(final D d){
            RealmResults<D> userList = (RealmResults<D>) mRealm.where(d.getClass()).findAll();
            return userList.get(userList.size()-1);
        }

        /**
         * 查找排序
         * */
        public <D extends RealmModel> void findAllSort(final D d){
            RealmResults<D> userList = (RealmResults<D>) mRealm.where(d.getClass()).findAll();
            RealmResults<D> result = userList.sort("age"); //根据age，正序排列
            result = userList.sort("age", Sort.DESCENDING);//逆序排列
        }

        /**
         * 根据name 获取value
         * */
        public <D extends RealmModel> D findAll(final D d,String name){
            RealmResults<D> userList = (RealmResults<D>)mRealm.where(d.getClass())
                    .isEmpty(name).findAll();
            return userList.get(userList.size()-1);
        }

### 升级数据库

       /**
         * 升级数据库
         * */
        public void updateRealm(int version){
            RealmConfiguration config = new RealmConfiguration.Builder()
                    .name("myrealm.realm") //文件名
                    .schemaVersion(version)
                    .migration(new CustomMigration())//升级数据库
                    .build();
        }

    public static  class CustomMigration implements RealmMigration {
            @Override
            public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {
                RealmSchema schema = realm.getSchema();
                if (oldVersion == 0 && newVersion == 1) {
                    RealmObjectSchema personSchema = schema.get("User");
                    //新增@Required的id
                    personSchema
                            .addField("id", String.class, FieldAttribute.REQUIRED)
                            .transform(new RealmObjectSchema.Function() {
                                @Override
                                public void apply(DynamicRealmObject obj) {
                                    obj.set("id", "1");//为id设置值
                                }
                            })
                            .removeField("age");//移除age属性
                    oldVersion++;
                }
            }
        }
