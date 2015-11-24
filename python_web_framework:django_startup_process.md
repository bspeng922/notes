## Django启动流程

1. 运行runserver命令来启动django项目：manager.py runserver 0.0.0.0:8080

manager.py代码
if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE",
                          "openstack_dashboard.settings")
    execute_from_command_line(sys.argv)