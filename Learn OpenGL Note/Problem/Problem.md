
---
***update : 2024. 12 . 15*** 
# `ImGui 控件鼠标事件锁定问题`

***Source :***
```if (ImGui::Checkbox(u8"开启FPS相机模式", &myCamera.cameraMode))
{
	if (myCamera.cameraMode)
	{
	    glfwSetCursorPosCallback(window, mouseCallBack);
	}
    else
    {
        glfwSetCursorPosCallback(window, nullptr);
	}
}
```

```
//mouseCallBack 函数实现
void mouseCallBack(GLFWwindow* window, double xposIn, double yposIn)
{
	if (!myCamera.cameraMode) 
	{
		// 如果没有启用FPS模式,跳过回调
		return;
	}
	float xpos = static_cast<float>(xposIn);
	float ypos = static_cast<float>(yposIn);

	if (myCamera.firstMouse)
	{
		lastMouseX = xpos;
		lastMouseY = ypos;
		myCamera.firstMouse = false;
	}
	//计算鼠标上一帧和当前帧的XY偏移量
	float xOffSet = xpos - lastMouseX;
	float yOffSet = lastMouseY - ypos;

	//更新鼠标位置
	lastMouseX = xpos;
	lastMouseY = ypos;
	
	////映射鼠标变量
	myCamera.ProcessMouseMovement(xOffSet, yOffSet, true);
}
```

***ProblemShow :***
![[imgui 控件鼠标事件被锁定.gif]]

***ProblemDes :***
我先是实现了一个 imgui 的复选框控件用于开启FPS相机模式,在我开启FPS相机模式再取消勾选后,鼠标点击事件就被锁定在这个控件内了,无法再使用其他的控件按钮或自由点击

***Solution:***
[[Solution#solution-1]]

---
***update : 2024. 12 . 15***
#
***Source :***
***ProblemShow :***
***ProblemDes :***
***Solution :***

---

