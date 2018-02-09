```python
# 1. 关于 employee_id 和 user_id 的迷思
# 代码来自于 hr_timesheet.py

"""
Summary: 用户界面数据模型使用的是 employee_id，为了从登录用户中获取到相关员工信息，使用以下的方式
"""

from odoo import api, fields, model

# codes ...

@api.model
def default_get(self, field_list):
    result = super(AccountAnalyticLine, self).default_get(field_list)
    # employee_id is not fetched from previous command
    if 'employee_id' in field_list and result.get('user_id'):
        result['employee_id'] = self.env['hr.employee'].search([('user_id', '=', result['user_id'])], limit=1).id
    return result

@api.onchange('employee_id')
def _onchange_employee_id(self):
    self.user_id = self.employee_id.user_id

# codes ...
```

```python
# 2. 关于 Transaction / Record 和 Balance / Summary 的迷思
# 代码来自于 project.py

"""
Summary: Balance / Summary 表中多个字段使用同一个 compute 方法，使用 One2many 的记录来计算值
"""

from odoo import api, fields, model

# codes ...

@api.depends('stage_id', 'timesheet_ids.unit_amount', 'planned_hours', 'child_ids.stage_id',
                 'child_ids.planned_hours', 'child_ids.effective_hours', 'child_ids.children_hours', 'child_ids.timesheet_ids.unit_amount')
def _hours_get(self):
    # logic code computing the value for fields
    pass

remaining_hours = fields.Float(compute='_hours_get', store=True, string='Remaining Hours', help="Total remaining time, can be re-estimated periodically by the assignee of the task.")
effective_hours = fields.Float(compute='_hours_get', store=True, string='Hours Spent', help="Computed using the sum of the task work done.")


# codes ...
```

```python
# 3. how to use with_context()

from odoo import api, fields, model

@api.model
def create(self, vals):
    context = dict(self.env.context)
    # Remove default_parent_id to avoid a confusion in get_record_data
    if context.get('default_parent_id', False):
        vals['parent_id'] = context.pop('default_parent_id', None)
        task = super(Task, self.with_context(context)).create(vals)
    return task
```



### 模型

#### 字段定义

- 加入 `domain` 来限定关联字段的可选值范围
- `Many2one` 和 `related=foo_id.bar_id` 同时存在，并指定 `readonly=True`，效果猜测为 `foo_id` 改变的时候，该字段会跟随变化，等同于普通的 `related` 字段，但是，同时建立了和 `bar` 模型的关系连接




### 视图

###### widgets

- `float_time` **hh:mm** 

###### field 属性

- `name`
- `ref`
- `widget`
- `position`
- `invisible`
- `sum`
- `readonly`
- `class`
- `attrs`
- `required`
- `string`
- `context`



###### tree 属性

- `default_order`
- `editable` **bottom | top**，适用于比较简单的模型，不用使用 `from` 视图来编辑
- `string`
- `import` **true | false**

###### menuitem 属性

- `groups`
- `web_icon`

###### state button

```xml
<button name="toggle_active" position="before">
    <button class="oe_stat_button" name="%(act_hr_timesheet_line_by_project)d" type="action" icon="fa-calendar" string="Timesheets" attrs="{'invisible': [('allow_timesheets', '=', False)]}" groups="hr_timesheet.group_hr_timesheet_user"/>
</button>
```



###### 带箭头指示的创建按钮的提示

```xml
<p class="oe_view_nocontent_create"> Click to create record.</p>
```

###### 下拉菜单项

- `name` 菜单项标题文本
- `src_model` 菜单在哪个模型视图中出现
- `res_model` 用以响应菜单操作的模型
- `view_mode` form / tree 视图中显示菜单项

```xml
<act_window id="project_task_action_view_timesheet"
            name="Timesheet Entries"
            src_model="project.task"
            res_model="account.analytic.line"
            view_mode="tree"
            view_id="hr_timesheet_line_tree"
            context="{
                'search_default_task_id': [active_id],
                'default_task_id': active_id,
            }"
        />
```

###### 继承

- `inherit_id` 属性

  ```xml
  <field name="inherit_id" ref="project.view_task_kanban"/>
  ```

- 可以使用多条命令改写、插入


- `QWeb` 语法
- `xpath` 的 `expr`
- `priority` 属性
- 元素的 `attrs`, `groups`


###### 模板

定义模板，用以 `controller` 中渲染页面。参考 `portal.py`