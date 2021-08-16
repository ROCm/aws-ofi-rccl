#
# Copyright (c) 2018-2019, Amazon.com, Inc. or its affiliates. All rights reserved.
# Modifications Copyright (c) 2021, Advanced Micro Devices, Inc. All rights reserved.
#
# See LICENSE.txt for license information
#

AM_CFLAGS = -g -O3 -Wall -fPIC -Wno-sign-compare
AM_CFLAGS += -I$(top_srcdir)/include -I/opt/cray/pe/mpich/8.1.9/ofi/cray/10.0/include
AM_LDFLAGS = -lmpi -lamdhip64 -L/opt/cray/pe/lib64
LDADD = ../src/librccl-net.la
CC=cc

bin_PROGRAMS = nccl_connection nccl_message_transfer ring
nccl_connection_SOURCES = nccl_connection.c
nccl_message_transfer_SOURCES = nccl_message_transfer.c
ring_SOURCES = ring.c
