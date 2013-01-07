---
layout: master
title: Linux input
---

## Overview

- input driver
- Input core
- Event handler

## Input Driver

### 分配、注册、注销input设备

	struct input_dev *input_allocate_device(void)
	int input_register_device(struct input_dev *dev)
	void input_unregister_device(struct input_dev *dev)

### 设置input设备 input_dev.

支持的事件类型、事件码、事件值的范围、input_id等信息


### 向子系统报告事件

	void input_event(struct input_dev *dev, unsigned int type, unsigned int code, int value)

## Reference

