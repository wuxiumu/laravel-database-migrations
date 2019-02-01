## 图

![图](/public/img/20190201111300.png)

## crate_users_table

```
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->integer('grade_id')->nullable()->index()->comment('积分等级');
            $table->decimal('total_consumption')->default(0)->comment('总消费');
            $table->integer('total_points')->default(0)->comment('累计积分');
            $table->string('mobile')->comment('手机号');
            $table->string('password')->nullable()->comment('密码');
            $table->string('wechat_openid')->default('')->comment('微信openid');
            $table->string('name')->default('')->comment('昵称');
            $table->string('avatar')->default('')->comment('头像');
            $table->string('realname')->default('')->comment('真实姓名');
            $table->string('sex')->default('')->comment('性别');
            $table->string('province')->default('')->comment('省');
            $table->string('city')->default('')->comment('市');
            $table->string('district')->default('')->comment('区');
            $table->string('address')->default('')->comment('地址');
            $table->string('birthday')->default('')->comment('生日');
            $table->string('email')->default('')->comment('email');
            $table->string('id_card')->default('')->comment('身份证号');
            $table->dateTime('last_actived_at')->nullable()->comment('最后活动时间');
            $table->tinyInteger('status')->default(1)->comment('状态  0 禁用 1 正常');
            $table->string('wechat_number', 45)->default('')->comment('微信号');
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();

            $table->index('wechat_openid');
            $table->index('mobile');
        });
    }
```            
## entrust_setup_tables
```
 public function up()
    {
        DB::beginTransaction();

        // Create table for storing roles
        Schema::create('roles', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('display_name')->nullable();
            $table->string('description')->nullable();
            $table->timestamps();
        });

        // Create table for associating roles to users (Many-to-Many)
        Schema::create('role_user', function (Blueprint $table) {
            $table->integer('user_id')->unsigned();
            $table->integer('role_id')->unsigned();

            $table->foreign('user_id')->references('id')->on('users')
                ->onUpdate('cascade')->onDelete('cascade');
            $table->foreign('role_id')->references('id')->on('roles')
                ->onUpdate('cascade')->onDelete('cascade');

            $table->primary(['user_id', 'role_id']);
        });

        // Create table for storing permissions
        Schema::create('permissions', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('display_name')->nullable();
            $table->string('description')->nullable();
            $table->timestamps();
        });

        // Create table for associating permissions to roles (Many-to-Many)
        Schema::create('permission_role', function (Blueprint $table) {
            $table->integer('permission_id')->unsigned();
            $table->integer('role_id')->unsigned();

            $table->foreign('permission_id')->references('id')->on('permissions')
                ->onUpdate('cascade')->onDelete('cascade');
            $table->foreign('role_id')->references('id')->on('roles')
                ->onUpdate('cascade')->onDelete('cascade');

            $table->primary(['permission_id', 'role_id']);
        });

        DB::commit();
    }

    /**
     * Reverse the migrations.
     *
     * @return  void
     */
    public function down()
    {
        Schema::drop('permission_role');
        Schema::drop('permissions');
        Schema::drop('role_user');
        Schema::drop('roles');
    }
```