# Hello World in Odoo 18 EE

This guide demonstrates how to create a basic "Hello World" module in Odoo 18 Enterprise Edition. We'll cover both backend and frontend development.

## Basic Module Structure

```
hello_world/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── hello_world.py
├── controllers/
│   ├── __init__.py
│   └── main.py
├── views/
│   ├── hello_world_views.xml
│   └── hello_world_templates.xml
├── security/
│   ├── ir.model.access.csv
│   └── hello_world_security.xml
└── static/
    ├── src/
    │   ├── js/
    │   │   └── hello_world.js
    │   └── xml/
    │       └── hello_world.xml
    └── description/
        └── icon.png
```

## 1. Module Declaration

### `__manifest__.py`
```python
{
    'name': 'Hello World',
    'version': '18.0.1.0.0',
    'category': 'Extra Tools',
    'summary': 'Basic Hello World Example',
    'description': """
        Hello World module for Odoo 18 Enterprise Edition
        ===============================================
        This module demonstrates basic Odoo development concepts.
    """,
    'author': 'Your Name',
    'website': 'https://www.example.com',
    'license': 'LGPL-3',
    
    'depends': ['base', 'web'],
    
    'data': [
        'security/hello_world_security.xml',
        'security/ir.model.access.csv',
        'views/hello_world_views.xml',
        'views/hello_world_templates.xml',
    ],
    
    'assets': {
        'web.assets_backend': [
            'hello_world/static/src/js/hello_world.js',
            'hello_world/static/src/xml/hello_world.xml',
        ],
    },
    
    'application': True,
    'installable': True,
    'auto_install': False,
}
```

### `__init__.py`
```python
from . import models
from . import controllers
```

## 2. Model Definition

### `models/hello_world.py`
```python
from odoo import models, fields, api

class HelloWorld(models.Model):
    _name = 'hello.world'
    _description = 'Hello World Message'
    
    name = fields.Char(string='Message', required=True)
    color = fields.Integer(string='Color Index')
    active = fields.Boolean(default=True)
    
    @api.model
    def say_hello(self):
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': 'Hello',
                'message': 'Welcome to Odoo 18!',
                'type': 'success',
            }
        }
```

## 3. View Definition

### `views/hello_world_views.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Form View -->
    <record id="view_hello_world_form" model="ir.ui.view">
        <field name="name">hello.world.form</field>
        <field name="model">hello.world</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <button name="say_hello" 
                            string="Say Hello" 
                            type="object" 
                            class="oe_highlight"/>
                </header>
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="color"/>
                        <field name="active"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <!-- Tree/List View -->
    <record id="view_hello_world_tree" model="ir.ui.view">
        <field name="name">hello.world.tree</field>
        <field name="model">hello.world</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="color"/>
            </tree>
        </field>
    </record>

    <!-- Search View -->
    <record id="view_hello_world_search" model="ir.ui.view">
        <field name="name">hello.world.search</field>
        <field name="model">hello.world</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <!-- Action -->
    <record id="action_hello_world" model="ir.actions.act_window">
        <field name="name">Hello World</field>
        <field name="res_model">hello.world</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create your first Hello World message!
            </p>
        </field>
    </record>

    <!-- Menu Items -->
    <menuitem id="menu_hello_world_root"
              name="Hello World"
              sequence="10"/>

    <menuitem id="menu_hello_world"
              parent="menu_hello_world_root"
              action="action_hello_world"
              sequence="10"/>
</odoo>
```

## 4. Security Configuration

### `security/ir.model.access.csv`
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hello_world_user,hello.world.user,model_hello_world,base.group_user,1,1,1,1
```

### `security/hello_world_security.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="hello_world_rule" model="ir.rule">
        <field name="name">Hello World Multi-Company Rule</field>
        <field name="model_id" ref="model_hello_world"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
</odoo>
```

## 5. OWL Component (Frontend)

### `static/src/js/hello_world.js`
```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Component } from "@odoo/owl";

class HelloWorldWidget extends Component {
    static template = "hello_world.HelloWorld";
    
    setup() {
        this.message = "Hello from OWL!";
    }
    
    onClick() {
        this.message = "Button clicked!";
        this.render();
    }
}

registry.category("hello_world_components").add("HelloWorld", HelloWorldWidget);
```

### `static/src/xml/hello_world.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hello_world.HelloWorld" owl="1">
        <div class="hello-world-widget">
            <h1><t t-esc="message"/></h1>
            <button class="btn btn-primary" t-on-click="onClick">
                Click Me!
            </button>
        </div>
    </t>
</templates>
```

## 6. Controller (Optional Web Route)

### `controllers/main.py`
```python
from odoo import http
from odoo.http import request

class HelloWorldController(http.Controller):
    @http.route('/hello', type='http', auth='public', website=True)
    def hello(self):
        return request.render('hello_world.hello_page', {
            'message': 'Hello from Odoo!'
        })
```

### `views/hello_world_templates.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="hello_page">
        <t t-call="web.layout">
            <div class="container">
                <h1><t t-esc="message"/></h1>
            </div>
        </t>
    </template>
</odoo>
```

## Installation and Usage

1. Place the module in your Odoo addons directory
2. Update the apps list in Odoo
3. Install the "Hello World" module
4. Access via menu: Hello World > Hello World

## Development Tips

1. **Debugging**
   ```python
   import logging
   _logger = logging.getLogger(__name__)
   _logger.info('Debug message')
   ```

2. **Module Upgrade**
   ```bash
   python3 odoo-bin -c odoo.conf -u hello_world
   ```

3. **Assets Generation**
   ```bash
   python3 odoo-bin -c odoo.conf --update hello_world --dev all
   ```

## Common Issues and Solutions

1. **Module Not Visible**
   - Check `__manifest__.py` is properly formatted
   - Update apps list in Odoo
   - Check addons path in configuration

2. **Access Rights Error**
   - Verify `ir.model.access.csv` entries
   - Check security rules
   - Verify user groups

3. **Asset Loading Issues**
   - Clear browser cache
   - Regenerate assets with `--dev all`
   - Check browser console for errors

## Next Steps

1. Add more complex models and relationships
2. Implement custom business logic
3. Add reports and wizards
4. Enhance the UI with advanced OWL components
5. Add automated tests

## Additional Resources

- [Odoo 18 Documentation](https://www.odoo.com/documentation/18.0/)
- [OWL Documentation](https://github.com/odoo/owl)
- [Odoo Development Cookbook](https://www.odoo.com/documentation/18.0/developer/howtos.html)
- [Odoo Community Association Guidelines](https://odoo-community.org/page/development)