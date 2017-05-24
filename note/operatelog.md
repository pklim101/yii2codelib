# yii2实现操作日志

## 一、创建migration.
```shell
php yii migrate/create user_log
```

## 二、修改数据文件
vim console/migrations/m150721_032220_admin_log.php
```php
<?php

use yii\db\Schema;
use yii\db\Migration;

class m150721_032220_admin_log extends Migration
{
    public function up()
    {
        $tableOptions = null;
        if ($this->db->driverName === 'mysql') {
            $tableOptions = 'CHARACTER SET utf8 COLLATE utf8_general_ci ENGINE=InnoDB COMMENT="后台操作记录"';
        }

        $this->createTable('{{%admin_log}}', [
            //'name'=>Schema::TYPE_STRING.'(200) PRIMARY KEY NOT NULL',
            'id'=>Schema::TYPE_PK,
            'admin_id'=>Schema::TYPE_INTEGER.'(10) UNSIGNED NOT NULL COMMENT "操作用户ID"',
            'admin_name'=>Schema::TYPE_STRING.'(200) NOT NULL COMMENT "操作用户名"',
            'addtime'=>Schema::TYPE_INTEGER.'(10) NOT NULL COMMENT "记录时间"',
            'admin_ip'=>Schema::TYPE_STRING.'(200) NOT NULL COMMENT "操作用户IP"',
            'admin_agent'=>Schema::TYPE_STRING.'(200) NOT NULL COMMENT "操作用户浏览器代理商"',
            'title'=>Schema::TYPE_STRING.'(200) NOT NULL COMMENT "记录描述"',
            'model'=>Schema::TYPE_STRING.'(200) NOT NULL COMMENT "操作模块（例：文章）"',
            'type'=>Schema::TYPE_STRING.'(200) NOT NULL COMMENT "操作类型（例：添加）"',
            'handle_id'=>Schema::TYPE_INTEGER.'(10) NOT NULL COMMENT "操作对象ID"',
            'result'=>Schema::TYPE_TEXT.' NOT NULL COMMENT "操作结果"',
            'describe'=>Schema::TYPE_TEXT.' NOT NULL COMMENT "备注"',

        ], $tableOptions);
    }

    public function down()
    {
        $this->dropTable('{{%admin_log}}');
    }
}
```

## 三、根据数据文件生成数据表
```shell
php yii migrate
```

## 四、创建操作记录的控制器、模型、视图

　　控制器
```php
<?php

namespace backend\controllers;

use backend\components\BaseController;
use common\models\AdminLog;
use yii;
use yii\data\ActiveDataProvider;

class AdminLogController extends BaseController
{
    public function actionIndex()
    {
        $dataProvider = new ActiveDataProvider([
            'query' => AdminLog::find(),
            'sort' => [
                'defaultOrder' => [
                    'addtime' => SORT_DESC
                ]
            ],
        ]);
        return $this->render('index',[
            'dataProvider' => $dataProvider
        ]);
    }

    public function actionView($id){
       return $this->render('view',[
           'model'=>AdminLog::findOne($id),
       ]);
    }

}
```

模型
```php
<?php

namespace common\models;

use Yii;

/**
 * This is the model class for table "{{%article}}".
 **/
class AdminLog extends \yii\db\ActiveRecord
{

    /**
     * @inheritdoc
     */
    public static function tableName()
    {
        return '{{%admin_log}}';
    }


    /**
     * @inheritdoc
     */
    public function attributeLabels()
    {
        return [
            'id'=>'操作记录ID',
            'title'=>'操作记录描述',
            'addtime'=>'记录时间',
            'admin_name'=>'操作人姓名',
            'admin_ip'=>'操作人IP地址',
            'admin_agent'=>'操作人浏览器代理商',
            'controller'=>'操作控制器名称',
            'action'=>'操作类型',
            'objId'=>'操作数据编号',
            'result'=>'操作结果',
        ];
    }



    public static function saveLog($controller ,$action,$result,$objId){
        $model = new self;
        $model->admin_ip = Yii::$app->request->userIP;
        $headers = Yii::$app->request->headers;
        $model->addtime = time();
        if ($headers->has('User-Agent')) {
            $model->admin_agent =  $headers->get('User-Agent');
        }
        $model->admin_id = Yii::$app->user->identity->id;
        $model->admin_name = Yii::$app->user->identity->email;

        $controllers = ['article','video','collection','collection-album','category','banner','exchange','user','admin'];
        if(!in_array(strtolower($controller),$controllers)) $controller = '';
        $actions = ['create','update','delete','login','logout'];
        if(!in_array(strtolower($action),$actions))$action = '';

        $model->controller = $controller;
        $model->action = $action;
        $model->result = $result;
        $model->objId = $objId;
        $model->title =  $model->admin_name.' '.$model->action.' '.$model->controller;
        $model->save(false);

    }
}
```

视图
```php
index视图

<?php

use yii\grid\GridView;

/* @var $this yii\web\View */
/* @var $dataProvider yii\data\ActiveDataProvider */

$this->title = '操作记录';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="handle-index">

    <?= GridView::widget([
        'dataProvider' => $dataProvider,
        'columns' => [
            'title',
            [
                'attribute'=>'addtime',
                'value'=>function($model){
                    return date('Y-m-d H:i:s',$model->addtime);
                },
            ],
            ['class' => 'yii\grid\ActionColumn','template'=>'{view}']
        ],
        'tableOptions'=>['class' => 'table table-striped']
    ]); ?>

</div>
```

```php
view视图

<?php

use yii\widgets\DetailView;

/* @var $this yii\web\View */
/* @var $model backend\models\Admin */

$this->title = '操作记录： '.$model->title;
$this->params['breadcrumbs'][] = ['label' => '操作记录', 'url' => ['index']];
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="admin-view">

    <?= DetailView::widget([
        'model' => $model,
        'attributes' => [
            'id',
            'admin_name',
            'addtime:datetime',
            'admin_ip',
            'admin_agent',
            'controller',
            'action',
            'objId',
            'result'
        ],
    ]) ?>

</div>
```

## 五、实现记录添加
```php
控制器中调用

public function actionCreate()
    {
        $model = new Banner();
        $model->status=Banner::STATUS_DISPLAY;
        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            //保存操作记录
            \common\models\AdminLog::saveLog('banner','create',$model->searchById($model->primaryKey),$model->primaryKey);

            Yii::$app->session->setFlash('success','Banner【'.$model->title.'】发布成功');
            return $this->redirect(['index']);
        } else {
            return $this->render('create', [
                'model' => $model,
            ]);
        }
    }

public function searchById($id){
    if (($model = Banner::findOne($id)) !== null) {
        return json_encode($model->toArray());
    } else {
        throw new \yii\web\NotFoundHttpException('The requested page does not exist.');
    }
}
```
