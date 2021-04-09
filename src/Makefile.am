#
# Copyright (c) 2018-2019, Amazon.com, Inc. or its affiliates. All rights reserved.
# Modifications Copyright (c) 2021, Advanced Micro Devices, Inc. All rights reserved.
#
# See LICENSE.txt for license information
#

AM_CFLAGS = -g -O3 -Wall -fPIC -Wno-sign-compare
AM_CFLAGS += -I$(top_srcdir)/include
AM_LDFLAGS = -lamdhip64

lib_LTLIBRARIES = librccl-net.la
librccl_net_la_SOURCES = nccl_ofi_net.c
